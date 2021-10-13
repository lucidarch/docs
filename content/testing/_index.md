---
title: "Testing"
date: 2021-10-13T19:11:14Z
draft: false
weight: 17
hide: ["header"]
head: "<hr />"
---

## Mocking Units

Lucid takes testing very seriously, thus it tries to make it extremely simple to simulate use cases with very little coding.

Every Lucid unit in the stack can be mocked by calling `mock([$args])->should*()` to make it easier to be replaced with a mock version of itself.

```php
GetUserByIDJob::mock(['id' => $id])->shouldReturn($user);
```

This will replace any call to `GetUserByIDJob` with the corresponding parameters with this instance and return the given `$user`,
as long as the passed arguments matches the unit's constructor signature, as well as the calling method

```php
$this->run(GetUserByIDJob::class, ['id' => $id]);
```

 otherwise an exception will be thrown for not finding a perfect match. For example, the following will certainly not pass the tests:

```php
$this->run(GetUserByIDJob::class, ['notid' => $id]);
```

The purpose of testing units is to make sure that whoever calls them is passing the right parameters
and will be aware of their signature changes, in which case tests shall fail.

For example, if `GetUserByIDJob` were to change signature and add another parameter, which ever test uses its mock shall fail:

```php
class GetUserByIDJob
{

    public function __construct($id, $isFlat)
    {
        ...
    }
}
```

Now our previous mock will certainly cause a failure since it's not matching the constructor args, and thus achieve a reliable suite of tests.

```php
GetUserByIDJob::mock(['id' => $id])->shouldReturn($user); // fail!
```

## Expectation Types

Creating a mock by calling `mock()` on a unit alone wouldn't suffice, there needs to be an expectation in order for the mock to be registered.

Since testing and mocking is all about simulating expectations, here are the methods you can use to set them.

### shouldBeDispatched
{{% panel %}}
#### `shouldBeDispatched(): void`

This method is used when we expect the unit to be dispatched but we're not waiting for any output.

**Example**

```php
$this->run(InviteMemberJob::class, ['memberId' => $memberId]);
```

```php
InviteMemberJob::mock(['memberId' => 'some-member'])->shouldBeDispatched();
```

{{% /panel %}}

### shouldReturn
{{% panel %}}

#### `shouldReturn($value): mixed`

This method is used to return any value that will be expected upon the unit's execution.

**Example**

```php
$repos = $this->run(FetchGitHubReposOperation::class, ['username' => $username]);
```

```php
$repos = collect(GitHubRepository::factory()->make(5));

FetchGitHubReposOperation::mock(['username' => $username])->shouldReturn($repos);
```

{{% /panel %}}

### shouldThrow
{{% panel %}}

**`shouldThrow($exception, $message = '', $code = 0, Exception $previous = null): void`**

This method simulates a unit throwing an exception.

**Example**

```php
$this->run(AddPostJob::class, ['title' => $title, 'content' => $content]);
```

```php
AddPostJob::mock(['title' => $title, 'content' => $content])
    ->shouldThrow(new NotLoggedInException(), 'you must be logged in to add a post', 401))
```

{{% /panel %}}

### shouldReturn[Bool]
{{% panel %}}

#### `shouldReturnTrue(): true` & `shouldReturnFalse(): false`

The use of these methods is as obvious, to return the corresponding boolean.

**Example**

```php
$authorized = $this->run(CheckUserAuthorizationJob::class, ['user' => $user]);
```

```php
CheckUserAuthorizationJob::mock(['user' => $user])->shouldReturnTrue(); // authorized

CheckUserAuthorizationJob::mock(['user' => $user])->shouldReturnFalse(); // not authorized
```

{{% /panel %}}

## Testing Features

Upon generating a feature, a test file would've already been generated with it in the same name.
Feature tests reside at `tests/Feature/*` in compliance with Laravel's directory structure.

When testing features we're interested in the sequence of executions that happen in the feature's `handle` method,
and in making sure that the right units (jobs and operations) are being called with the correct arguments.
To achieve that we'd have to mock some calls and execute the ones we can.

Lucid makes it extremely easy to mock and set expectations to replace the units you need.

Consider a feature to create a channel in a chat application; the following steps need to be taken in order to successfully create one:

1. Fetch the member's object by ID
2. Make sure the user is authorized to create a channel
3. Create channel
4. Add the user to the channel
5. Invite members to join the channel as well

Here's how that might look like in code (simplified):

```php
class CreateChannelFeature extends Feature
{
    public function handle(CreateChannelRequest $request)
    {
        $member = $this->run(GetMemberByIdJob::class, ['id' => $request->input('id')]);

        $authorized = $this->run(AuthorizeMemberActionJob::class, [
            'action' => Action::CREATE_CHANNEL,
        ]);

        if (!$authorized) {
            throw new UnauthorizedActionException();
        }

        $channel = $this->run(CreateChannelJob::class, ['title' => $request->input('title')]);

        $this->run(AddMemberToChannelJob::class, [
            'member'  => $member,
            'channel' => $channel,
        ]);

        $this->run(InviteMembersToChannelOperation::class, [
            'channel' => $channel,
            'members' => $request->input('invited'),
        ]);
    }
}
```

When testing that feature class, we will use Lucid's unit mocking techniques to turn some knobs around and simulate our test cases.
Here are a few examples:

**Test Unauthorized Member**

To simulate an unauthorized user we'd need to mock `AuthorizeMemberActionJob` and return `false` so that it throws the corresponding exception.

```php
public function test_create_organization_unauthorized_member()
{
    $member = Member::factory()->make();

    AuthorizeMemberActionJob::mock(['action' => Action::CREATE_CHANNEL])
        ->shouldReturnFalse();

    $this->expectException(UnauthorizedActionException::class);

    $this->postJson('/channels', [
        'id' => $member->id,
        'title' => 'ping-channel',
        'invited' => ['member-id-1', 'member-id-2']
    ]);
}
```

**Simulate an Invitation Error**

To see how our code will behave when an issue occurs as we invite other members, let's cause it to happen.

```php
public function test_create_organization_unauthorized_member()
{
    $member = Member::factory()->make();
    $channel = Channel::factory()->make();

    AuthorizeMemberActionJob::mock(['action' => Action::CREATE_CHANNEL])
        ->shouldReturnTrue();

    // we will need this to be pass it to the following operation
    CreateChannelJob::mock(['title' => $channel->title])
        ->shouldReturn($channel);

    InviteMembersToChannelOperation::mock([
        'channel' => $channel,
        'members' => ['member-id-1', 'member-id-2']
    ])->shouldThrow(new DatabaseConnectionException(), 'could not connect to database', 500);

    $this->postJson('/channels', [
        'id' => $member->id,
        'title' => 'ping-channel',
        'invited' => ['member-id-1', 'member-id-2']
    ]);
}
```

## About Testing and Databases

It is recommended that Feature tests cover the entire set of functionalities, including storage,
however unit tests that cover Jobs and Operations should be left for preference, though make sure you are consistent across your codebase.

Nevertheless, sometimes we may choose to not hit the DB. Mocking can help with that since it will replace
the unit (job/operation) with its mock so when executing its code won't run. It is extremely important, however, to ensure that the job
being mocked has tests of its own, extensively, otherwise we'll be leaving loose ends and may face unanticipated outcomes.

