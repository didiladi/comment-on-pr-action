![build-test](https://github.com/didiladi/comment-on-pr-action/workflows/build-test/badge.svg)

# Use Case

This action enables you to automate running tests which need access to secrets in a secure manner for forks. Consider this open-source workflow:
* Your repository is forked and a PR is created
* For merging, the maintainers need to check whether the integration tests pass
  * Sadly, the integration tests need to access secrets - we can't run the integration tests in the context of the fork
  * By doing so, we would bleed the secrets into the fork
  * So, what do we do? :question:

## The solution

* :+1: A maintainer performs a code review - the code looks awesome!
* :label: The maintainer adds a label `run-integration-tests` to the PR
* The cron-triggered workflow using the [trigger-workflow-for-pr](https://github.com/didiladi/trigger-workflow-for-pr-action) action runs and picks up the new PR.
* :envelope: Since the label `run-integration-tests` is present, and there is no comment yet from this action, an event `trigger-integration-tests` is dispatched.
* :inbox_tray: There is a workflow in the upstream repository which is triggered by the event.
* :heavy_check_mark: It runs the integration tests and wites back the results to the PR. Of course, the tests succeed.
* :100: Now, the maintainer can merge the awesome PR
<br/><br/>

This repository contains the code for the action to comment on the PR in the scenario above.

To see a full working example of the workflow, please look at the [didiladi/mac-build-test](https://github.com/didiladi/mac-build-test) repository.

---

## Comment on a PR

Use this action to comment on an PR, if the fork workflow was triggered by action [@didiladi/trigger-workflow-for-pr](https://github.com/didiladi/trigger-workflow-for-pr-action):

You are asking: Why do I need this special action for commenting on the PR when using [@didiladi/trigger-workflow-for-pr](https://github.com/didiladi/trigger-workflow-for-pr-action)?

The answer is easy: This action embeds a hidden sting into the comments, which gets picked up by [@didiladi/trigger-workflow-for-pr](https://github.com/didiladi/trigger-workflow-for-pr-action) to avoid adding the same comment multiple times on the PR.


### Usage

Add a comment to the PR with the status of the workflow:

```
- name: Setup Message
  id: setup_message
  if: ${{ always() }}
  shell: bash
  env:
    STATUS: "${{ job.status }}"
  run: |
    if [[ $STATUS == "success" ]]; then
      echo "::set-output name=MESSAGE::### 🎉 Integration tests ran successfully on ${{ matrix.os }} 🥳"
    else
      echo "::set-output name=MESSAGE::### ❌ Integration tests failed on ${{ matrix.os }}"
    fi

- name: Post Comment
  if: ${{ always() }}
  uses: didiladi/comment-on-pr-action@v1
  with:
    pr-id: ${{ github.event.client_payload.pr_number }}
    comment-prefix: ${{ github.event.client_payload.comment_prefix }}
    message: ${{ steps.setup_message.outputs.MESSAGE }}
    token: ${{ secrets.GITHUB_TOKEN }}
    user: didiladi
    repo: ${{ github.event.client_payload.repo }}
```

### Input variables

* pr-id:
  * required: true
  * description: the number of the pull request to comment on.
* comment-prefix:
  * required: true
  * description: the hidden prefix contained in the comment (this usually comes from action [@didiladi/trigger-workflow-for-pr](https://github.com/didiladi/trigger-workflow-for-pr-action))
* message:
  * required: true
  * description: the message to post on the pull request
* token:
  * required: true
  * description: the token (in most cases this is `secrets.GITHUB_TOKEN`). If you want a different user to comment on the PR, please use a dedicated PAT.
* user:
  * required: true
  * description: the owner of the upstream repository (not the fork)
* repo:
  * required: true
  * description: the reporitory name for the pull request

### Output variables

* url: the html url of the resulting comment 