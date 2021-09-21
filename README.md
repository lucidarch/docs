<p align="center"><a href="https://lucidarch.dev" target="_blank"><img src="https://raw.githubusercontent.com/lucidarch/artwork/main/logo.jpg" width="400"></a></p>

<p align="center" style="margin-left: -20px">
    <a href="https://docs.lucidarch.dev"><img src="http://img.shields.io/badge/read_the-docs-2196f3.svg" alt="Documentation"></a>
    <a href="https://lucid-slack.herokuapp.com"><img src="https://lucid-slack.herokuapp.com/badge.svg" alt="Slack Chat"/></a>
    <a href="https://github.com/lucidarch/lucid/actions?query=workflow%3Atests"><img src="https://github.com/lucidarch/lucid/workflows/tests/badge.svg" alt="Build Status"></a>
    <a href="https://packagist.org/packages/lucidarch/lucid"><img src="https://img.shields.io/packagist/v/lucidarch/lucid" alt="Latest Stable Version"></a>
    <a href="https://github.com/lucidarch/lucid/blob/main/LICENSE"><img src="https://img.shields.io/packagist/l/lucidarch/lucid" alt="License"></a>
</p>


# Documentation
The repository of [docs.lucidarch.dev](https://docs.lucidarch.dev).

### Run Docs Locally
- Clone this repository
- In the root of the project, use `docker-compose up -d` to run the server
- Visit `http://localhost:1313` to see it

### Contribute
The docs are built using [Hugo](https://gohugo.io) with a customised version of [docport theme](https://docport.netlify.app).

- Fork this repository
- Create a branch for your contribution `git checkout fix/issue-32`
- Run containers uring `docker-compose up -d`
- Generate a new `section/_index.md` file using `docker-compose exec hugo hugo new {section}/_index.md` (yes two hugos) or edit an existing one
- Commit your changes with a meaningful short message
- Push the code to your repository `git push origin fix/issue-32`
- Open a [pull request](https://github.com/lucidarch/docs/compare) in this repository to the `main` branch

### Structure

- Hugo configuration is in `config.toml` as indicated in [the docs](https://gohugo.io/getting-started/configuration/)
- Every section in the sidebar is a directory with its name in `snake-case` under `content`, and an `_index.md` file at its root
- The head of `_index.md` is the definition of the section
    ```
    ---

    title: "CLI Reference"
    date: 2020-10-28T17:04:19Z
    draft: false
    weight: 15
    hide: ["header"]
    head: "<hr />"


    ---
    ```

    - `date` will be taken care of by the engine when generating a new file with `hugo new`
    - `weight` organizes section in their order
    - `hide` things to hide when the section is visited. Options are: `header` and `toc`
    - `head` only present for sections that need something in their head, absent in the others that don't.
