# Branch and PR previews

Branch and PR previews are automatically deployed by [Heroku](https://www.heroku.com/).

Note that pull requests should be made to the `LandRegistry/hmlr-design-system` base repository and not the default Github repository.

## Pull Request previews

Previews of Pull Requests are automatically published to a URL which has the
prefix `hmlr-design-system-pr-` followed by the identifier number of the pull request.

For example, pull request #137 would be deployed to
`hmlr-design-system-pr-137.herokuapp.com`.

The Heroku bot should comment on each PR shortly after building with a link to
the preview.

## Branch previews

When a new branch is pushed to GitHub a preview website will be deployed.
Branch deploys are published to a URL which includes the branch name as a prefix.

For example, if a branch is called `staging`, it will deploy to `hmlr-design-system-staging.herokuapp.com`.

## Configuration

The Heroku account is tied to the hmlr-design-system(?) GitHub user, the credentials for which can be found ...
