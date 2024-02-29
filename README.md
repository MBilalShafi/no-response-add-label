# No response with add back a label

A GitHub Action that closes Issues where the author hasn't responded to a request for more information.

It was forked from https://github.com/lee-dohm/no-response
On top of the functionality offered in the original action, this one also adds back a label on the removal of the `responseRequiredLabel`

## Use

Recommended basic configuration:

```yaml
name: No Response

# `issues`.`closed`, `issue_comment`.`created`, and `scheduled` event types are required for this Action
# to work properly.
on:
  issues:
    types: [closed]
  issue_comment:
    types: [created]
  schedule:
    # Schedule for five minutes after the hour, every hour
    - cron: '5 * * * *'

jobs:
  noResponse:
    runs-on: ubuntu-latest
    steps:
      - uses: MBilalShafi/no-response-add-label@v0.0.6
        with:
          token: ${{ github.token }}
```

Another example with further configurations:

```yaml
name: No Response

# `issues`.`closed`, `issue_comment`.`created`, and `scheduled` event types are required for this Action
# to work properly.
on:
  issues:
    types: [closed]
  issue_comment:
    types: [created]
  schedule:
    # Schedule for five minutes after the hour, every hour
    - cron: '5 * * * *'

jobs:
  noResponse:
    runs-on: ubuntu-latest
    steps:
      - uses: MBilalShafi/no-response-add-label@v0.0.6
        with:
          token: ${{ github.token }}
          # auto close issues with no response from author for 7 days
          daysUntilClose: 7
          # label to use to track when a response is required (once a response is made, this label will be auto removed)
          # if the label doesn't exist, it will be auto-created
          responseRequiredLabel: "status: waiting for author's response"
          # once the responseRequiredLabel is removed, this one will be added for tracking on maintainers' side
          optionalFollowupLabel: "status: waiting for maintainer's response"
          # this comment will be posted when closing the issue
          closeComment: >
            The issue has not been closed due to no response from the author, feel free to reopen
```


### Inputs

See [`action.yml`](action.yml) for defaults.

```tsx
// Markdown text to post as a comment when an issue is going to be closed. Set to `false` to disable commenting when closing an issue.
closeComment?: string;

// Number of days to wait for a response from the original author before closing.
daysUntilClose?: number;

// Text of the label used to indicate that a response from the original author is required.
// @default 'needs-more-information'
responseRequiredLabel?: string;

// Color for the `responseRequiredLabel`, encoded as a hex string. **Only** used when creating the label if it does not already exist.
// @default 'ffffff'
responseRequiredColor?: string;

// Text of the label that will be added back when the `responseRequiredLabel` is removed due to author responding back.
optionalFollowupLabel?: string;

// Color for the `optionalFollowupLabel`, encoded as a hex string. **Only** used when creating the label if it does not already exist.
// @default 'ffffff'
optionalFollowupLabelColor?: string;

// Token used to access repo information. The default GitHub Actions token is sufficient.
token: string;
```

### Outputs

None.

## Action flow

The intent of this Action is to close issues that have not received a response to a maintainer's request for more information. Many times issues will be filed without enough information to be properly investigated. This Action allows maintainers to label an issue as requiring more information from the original author. If the information is not received in a timely manner, the issue will be closed. If the original author comes back and gives more information, the label is removed and the issue is reopened, if necessary.

### Scheduled

At the scheduled times, it searches for issues that are:

- Open
- Have a label named the same as the `responseRequiredLabel` value in the configuration
- The `responseRequiredLabel` was applied more than `daysUntilClose` ago

For each issue found, it:

1. If `closeComment` is not `false`, posts the contents of `closeComment`
2. Closes the issue

### `issue_comment` Event

When an `issue_comment` event is received, if all of the following are true:

- The author of the comment is the original author of the issue
- The issue has a label named the same as the `responseRequiredLabel` value in the configuration

It will:

1. Remove the `responseRequiredLabel`
2. Reopen the issue if it was closed by someone other than the original author of the issue
3. If `optionalFollowupLabel` is passed in the config, add it to the issue

### `issues` Event

When an `issues` event is received, if all of the following are true:

- The issue was closed (i.e. `action: closed`)
- The user which closed the issue is the same as the original author of the issue
- The `optionalFollowupLabel` is passed in the config, and added as a label to the issue

It will remove the `optionalFollowupLabel`

## License

[MIT](LICENSE.md)
