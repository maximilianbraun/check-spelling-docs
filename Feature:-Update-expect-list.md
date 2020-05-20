# Update expect list

Presently, the action reports a comment which includes a posix shell command to update the expect list.

It would be much nicer if one could click something and have the action update the expect list.

### Comment limit

GitHub returns an error if the comment is too long:
> "body is too long (maximum is 65536 characters)"

This functions as a limit on the number of words reported initially.

Also, duplicating the words to expect list is just ugly.

Once the comment is effectively self-aware, it makes sense for the command to retrieve the bullet list instead of encoding the words twice.

## User trigger implementation paths

### CheckRunEvent: action

GitHub's [CheckRunEvent: action](https://developer.github.com/v3/activity/events/types/#checkrunevent-api-payload)
offers a way to implement fix-it functionality.

I'm not sure if that's available for GitHub Actions. I'm hoping it does

⛈ I couldn't find a way to do this
### Reaction handling

It'd be really shiny if someone could click the 😶 and select the 🚀 [Reaction](https://developer.github.com/v3/reactions/) to trigger this.

⛈ Unfortunately, the [Reactions preview](https://developer.github.com/changes/2016-05-12-reactions-api-preview) doesn't appear to support this functionality.

### Comment handling

It's definitely possible for the bot to respond to a comment.
1. 🤖 posts a comment (as it does today)
1. 🤖 gets its url (🌟 it doesn't use the url today...)
1. 🤖 update the comment with the url and instructions saying roughly "reply quoting this line" 
1. 🤺 user performs the action above

   🔨 I've started the work for this.
1. 🤖 responds to the action by retrieving the comment (based on the url which is encoded in the reply)
1. ➡ proceed to work


## Commit implementation paths

I'm tentatively planning to retrieve the shell script from the comment, run its equivalent, git add/remove, commit.

1. The command (which used to be **posix shell**) is now [[Perl|Feature: Other shells]]
1. With the command retrieved from the comment
1. Perform the command equivalent (the command will be refactored so that the same code can be shared between the two code paths)
1. I don't intend to delete empty files, as they represent structure even if they're temporarily empty.
1. Add the changed files
1. Create a commit:

   * **Committer** will be the bot
   * **Author** will be the user who commented (need to get username + email address)
   * **Date** will be the comment's date
   * Message will be something like:

     ```
     Applying automated check-spelling metadata updates

     Commands as per <bot-comment-url>
     Accepted in <user-comment-url>
     ```

### Git push

The "easiest path" is to push straight to the branch

### Create a PR for the PR / branch

I may need to add fallback code whereby I create / recycle a branch (based on the name of the PR/branch) and then open a PR against the original branch.

## Downsides

I see a lot of people running the commands blindly and missing the fact that there are real misspellings in the output.

* Part of this is that they aren't used to looking at the output.
* Part of this is that the current release (0.0.15-alpha) doesn't highlight certain cases (this is fixed in a prerelease, I expect it to be available in 0.0.16-alpha shortly).