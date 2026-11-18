[![StepSecurity Maintained Action](https://raw.githubusercontent.com/step-security/maintained-actions-assets/main/assets/maintained-action-banner.png)](https://docs.stepsecurity.io/actions/stepsecurity-maintained-actions)

# Commit Comment
[![CI](https://github.com/step-security/commit-comment/workflows/CI/badge.svg)](https://github.com/step-security/commit-comment/actions?query=workflow%3ACI)

A GitHub action to create a comment for a commit on GitHub.

![Commit Comment Example](https://github.com/step-security/commit-comment/blob/main/comment-example.png?raw=true)

## Usage

### Add a comment to the current context's commit SHA

The SHA defaults to `github.sha` OR, for `pull_request` events `github.event.pull_request.head.sha`.

```yml
      - name: Create commit comment
        uses: step-security/commit-comment@v4
        with:
          body: |
            This is a multi-line test comment
            - With GitHub **Markdown** :sparkles:
            - Created by [commit-comment][1]

            [1]: https://github.com/step-security/commit-comment
          reactions: '+1'
```

### Update a commit comment

```yml
      - name: Update commit comment
        uses: step-security/commit-comment@v4
        with:
          comment-id: 557858210
          body: |
            **Edit:** Some additional info
          reactions: eyes
```

### Add commit comment reactions

```yml
      - name: Add reactions
        uses: step-security/commit-comment@v4
        with:
          comment-id: 557858210
          reactions: |
            heart
            hooray
            laugh
```

## Action inputs

| Name | Description | Default |
| --- | --- | --- |
| `token` | `GITHUB_TOKEN` or a `repo` scoped [PAT](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token). | `GITHUB_TOKEN` |
| `repository` | The full name of the target repository. | `github.repository` (current repository) |
| `sha` | The commit SHA. | `github.sha` OR, for `pull_request` events `github.event.pull_request.head.sha` |
| `path` | Relative path of the file to comment on. | |
| `position` | Line index in the diff to comment on. | |
| `comment-id` | The id of the comment to update. | |
| `body` | The comment body. Cannot be used in conjunction with `body-path`. | |
| `body-path` | The path to a file containing the comment body. Cannot be used in conjunction with `body`. | |
| `edit-mode` | The mode when updating a comment, `replace` or `append`. | `append` |
| `append-separator` | The separator to use when appending to an existing comment. (`newline`, `space`, `none`) | `newline` |
| `reactions` | A comma or newline separated list of reactions to add to the comment. (`+1`, `-1`, `laugh`, `confused`, `heart`, `hooray`, `rocket`, `eyes`) | |
| `reactions-edit-mode` | The mode when updating comment reactions, `replace` or `append`. | `append` |

Note: In *public* repositories this action does not work in `pull_request` workflows when triggered by forks.
Any attempt will be met with the error, `Resource not accessible by integration`.
This is due to token restrictions put in place by GitHub Actions. Private repositories can be configured to [enable workflows](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#enabling-workflows-for-forks-of-private-repositories) from forks to run without restriction. See [here](https://github.com/step-security/create-pull-request/blob/main/docs/concepts-guidelines.md#restrictions-on-repository-forks) for further explanation. Alternatively, use the [`pull_request_target`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request_target) event to comment on pull request commits.

#### Outputs

The ID of the created comment will be output for use in later steps.
Note that in order to read the step output the action step must have an id.

```yml
      - name: Create commit comment
        uses: step-security/commit-comment@v4
        id: cc
        with:
          body: |
            My comment
      - name: Check outputs
        run: |
          echo "Comment ID - ${{ steps.cc.outputs.comment-id }}"
```

### Setting the comment body from a file

```yml
      - name: Create commit comment
        uses: step-security/commit-comment@v4
        with:
          body-path: 'comment-body.md'
```

### Using a markdown template

In this example, a markdown template file is added to the repository at `.github/comment-template.md` with the following content.
```
This is a test comment template
Render template variables such as {{ .foo }} and {{ .bar }}.
```

The template is rendered using the [render-template](https://github.com/chuhlomin/render-template) action and the result is used to create the comment.
```yml
      - name: Render template
        id: template
        uses: step-security/render-template@v1
        with:
          template: .github/comment-template.md
          vars: |
            foo: this
            bar: that

      - name: Create commit comment
        uses: step-security/commit-comment@v4
        with:
          body: ${{ steps.template.outputs.result }}
```

### Accessing commits and comments in other repositories

You can create and update commit comments in another repository by using a [PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) instead of `GITHUB_TOKEN`.
The user associated with the PAT must have write access to the repository.

## License

[MIT](LICENSE)
