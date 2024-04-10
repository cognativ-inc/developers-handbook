# Git guidelines

Git is an essential skill for all Cognativ developers, and proficiency in using git through platforms such as Bitbucket, GitHub, GitLab or through the Terminal is crucial or the websites itself is a must. As our company grows, it's essential to maintain stability in our codebases and establish a standard process for “git flows.” This chapter will provide guidelines for working with git in any project.

It's important to note that some projects may be managed on clients' git accounts, and in those cases, these guidelines may not be applicable. However, it's worth noting that the guidelines provided are based on best practices used by top technology companies worldwide, and may align with the guidelines of our clients.

## Communicate effectively

Good communication is key to a successful collaboration process, and believe me, once everyone knows how to do it you will find out browsing and working with others teams will be seamlessly. Be sure to let your team members know when you're working on something, what you're working on, and when you're done. Also, make sure to respond to any questions or concerns in a timely manner.

English is the *default* language for any communication, both on channels, Jiras or Bitbucket. Same as in the programming and general developer guidelines in this handbook, we advise to not use your local language, it may confuse others.

Remember:

1. Always communicate in English.
2. Use neutral language, avoid feelings.
3. Be short, concise, don’t write wall of texts

***

## The structure of every Git project

There are many different branch structures that can be used for projects managed by Git, and the best one will depend on the specific needs of the project and the team working on it. However, one commonly used and effective branch structure is the GitFlow workflow and this will be our standard at Cognitive, unless it's a client specific git repository or client needs imposed the git architecture.

GitFlow consists of two main branches: "develop" and "master". The "develop" branch is where new features and changes are added, and is where pull requests are made and code is reviewed. The "master" branch is where stable, production-ready code is kept.

Additionally, GitFlow also includes feature branches and release branches. Feature branches are used to implement new features, and are branched off of "develop" and merged back into it after review. Release branches are used to prepare for a release, and are branched off of "develop" and merged into "master" and "develop" after the release.

This structure allows for a clear separation of development and production code, and provides a consistent, organized workflow for making changes and releasing new versions of the software.

In the GitFlow workflow, there are several different types of branches that may be used, each with their own purpose:

* **master:** This branch contains the production-ready code that is currently live. Only release branches or hotfix branches should be merged into the master branch.

* **develop:** This branch is where new features and changes are added, and is where pull requests are made and code is reviewed. All feature branches are merged into the develop branch once they are ready for review.

* **feature:** These branches are used to implement new features and are branched off of "develop". They are used to make changes to the codebase and should be regularly merged back into the "develop" branch for review and testing.

* **release:** These branches are used to prepare for a release and are branched off of "develop". They are used to stabilize the codebase and fix any last-minute bugs before the release.

* **hotfix:** These branches are used to fix critical bugs in the production code. They are branched off of "master" and should be used to quickly fix any issues that are affecting users. Once the hotfix is complete, the hotfix branch should be merged into both "master" and "develop".

***


## Pull Requests

> **Pull requests are mandatory.** **Never commit or push to main branches.**

If someone makes commits directly to a development branch without creating pull requests, it could cause issues with code review and collaboration.

Pull requests allow for multiple people to review and discuss code changes before they are merged into the development branch, which helps ensure that the code is of high quality and that any potential conflicts with other changes are resolved. 

Additionally, making commits directly to the development branch may not be in line with the team's workflow or development process, and could lead to confusion and miscommunication.

Committing to the developer branch while others are creating pull requests (PRs) from that branch can cause conflicts if the same lines of code are changed by multiple people. To avoid these conflicts, it is best practice to avoid committing to main branches, and to regularly pull the latest changes from the branch before pushing your own changes. Additionally, it is best practice to use feature branches for new code development and avoid committing directly to the develop branch. This will minimize the chance of merge conflicts.

### Naming branches
 
Creating meaningful branch names can help to organize and identify the purpose of different branches in your codebase.

Bad naming:

> branch1

> feature/fixes

> refactor-old-code

Good naming:

> feature/new-login-ui

> feature/refactor-css-new-ui

> bug/fixed-margin-login-page

Here are a few tips for creating meaningful branch names:

* Feature branches should be named with a prefix of "feature/", such as "feature/new-login-system”, “feature/voting-on-posts”, etc.
* Release branches should be named with a prefix of "release/", such as "release/v1.2". Once the release is ready, the release branch will be merged into both "master" and "develop".
* Hotfix branches should be named with a prefix of "hotfix/", such as “hotfix/login-issue”.

Adhering to these guidelines is crucial to ensure the integrity and success of both the company and the work of your colleagues. Failure to do so can put both at risk.

### Creating Pull Requests

There are conventions and keywords that can be used in pull request titles and descriptions to indicate the status or nature of the changes being made. This helps the team to review your code and speed up the process.

#### Writing good PR titles

**"[WIP]" stands for "Work in Progress"** and is used to indicate that a pull request is not yet ready for review or merging. It is commonly used when a developer has started working on a new feature or bug fix, but is not yet finished. Adding "[WIP]" to the title of the pull request can help prevent it from being accidentally reviewed or merged before it is ready.

> Ex. [WIP]: feature/Group management for group administrators

**"[RFC]" (Request for Comments):** Indicates that the pull request is a proposal for a new feature or change, and is being used to gather feedback before the work is completed.

**"[BUG]" (Bug fix):** Indicates that the pull request is for a bug fix, and can help make it clear that the changes are related to resolving a specific issue.

**"[HOTFIX]" (Hotfix):** Indicates that the pull request is for a critical bug fix that needs to be addressed as soon as possible.

**"[IMPROVEMENT]" (Improvement):** Indicates that the pull request is for an improvement or enhancement to an existing feature, rather than a new feature or bug fix.

**"[PERFORMANCE]" (Performance):** Indicates that the pull request is related to performance optimization.

**"[DOCUMENTATION]" (Documentation):** Indicates that the pull request is related to documentation update or additions.

Using these conventions can help make it clear what the pull request is for and its priority level, making it easier for team members to understand and review the changes.

#### Writing good PR descriptions

Writing clear and informative descriptions in pull requests (PRs) is important to help others understand the changes that have been made, and to facilitate the review and merging process. Here are a few tips for writing effective PR descriptions:

1. **Summarize the changes:** Start the description with a brief summary of the main changes that have been made in the branch. This will give reviewers a quick overview of what to expect.
2. **Provide context:** Explain the background and reasoning behind the changes. This will help reviewers understand the motivation and goals of the work.
3. **Include screenshots and gifs:** If the changes are visual, include screenshots or gifs to help reviewers understand the changes more easily.
4. **List any dependencies:** If the changes are dependent on other changes or features, list them so that reviewers can check that everything is in place before merging.
5. **Link to relevant issues or tasks:** If the changes are related to a specific issue or task, include a link to it in the description.
6. **Explain testing:** Explain how you have tested the changes, and what testing still needs to be done.
7. **Be clear and concise:** Keep the description clear and to the point. Avoid using jargon or technical terms that others may not understand.
8. **Clearly specifying the technical requirements** in your code can prevent confusion and unnecessary reviews. By stating the programming language, framework, or version of ECMAScript that is being used, reviewers can ensure they are equipped to properly evaluate the code. This will help to avoid any confusion and zensure a more efficient review process.

By following these tips, you can write clear and informative PR descriptions that will help reviewers understand the changes that have been made, and facilitate the review and merging process.

### Merge often

Regularly merge and rebase: Regularly merge the development branch into feature branches and rebase feature branches onto the development branch to ensure that the branches stay in sync and that conflicts are resolved.

### Size of a branch, 

A common issue faced by development teams is the creation of large branches containing a significant amount of code, resulting in lengthy pull requests that can be challenging to review. To address this problem, it's essential to consider the size of the feature being developed.

For instance, a registration feature may involve changes to the user interface, interactions, copy, translations, state management, and error handling. Instead of creating a single branch that includes all of these changes, it's better to break it down into smaller, manageable chunks that can be reviewed more efficiently. There is no specific number of commits or lines of code that dictate when to split a branch, but it's crucial to think about the complexity and size of the feature being developed.

### When working on features, avoid refactoring

It's important *to avoid making significant refactors while working on new features*. Refactoring a lot of code or introducing new libraries can cause issues if not all changes are made correctly, potentially breaking the entire application. Instead, it's best to create a separate branch for refactors and another for new features. This way, code review can be done separately and more efficiently.

For example, if you want to implement a feature that allows users to vote, it's best to avoid making significant changes to the CSS classes throughout the application, such as renaming them, at the same time. This can create unnecessary noise and consume time for those reviewing the code. It's better to make a separate branch for refactoring and another for the feature development. This will make code review more manageable and less time-consuming for your colleagues.

If when working on features it happens you need a refactor, then leave the feature alone, make a new branch for refactor and work on that, then come back and merge in your feature branch all the refactors.

## Code reviews

Code review is a powerful tool for creating high-quality software at Cognativ. It provides a way to ensure that the code is of high quality, and any potential issues are identified and resolved early on. Code reviews are mandatory at Cognativ, and every developer is expected to participate, regardless of whether they are assigned to a particular project or not. By using code reviews as a way to catch issues early, we can work together as a team to develop great software.

### Bests practices

**Assigning a primary reviewer:** Assign a primary reviewer for each pull request to ensure that someone is responsible for leading the review process. You can tag them if you want their opinion. This can be optional when what are you changing is not modifying the behavior of the application dramatically. But when you do a big refactor, or you change the code significantly, it's work 

**The most efficient way to request a code review is to share the pull request (PR) in the t-front-back-dev slack channel** and explicitly ask for a review. This method ensures that the review process is completed in a timely manner.

**Use automated tools:** Use automated tools to check for common issues, such as style inconsistencies and potential bugs, etc.

**Encourage open communication:** Encourage open communication between reviewers and the person who submitted the code, and foster a culture of constructive feedback.

**Review the code as soon as possible:** Review the code as soon as possible to catch issues early and prevent delays in the development process.

**When reviewing others' work, it's important to be mindful of the language and tone used.** The goal is to help colleagues improve the quality of their code, not to undermine their competence. Avoid using negative language or emojis that express negative emotions. Instead, use language that is constructive and aimed at helping the colleague improve their code. Emojis should only be used for greetings or congratulating a job well done. Remember to always be respectful and professional when providing feedback.

A "do not" example on code reviews:

> What is this? 🤮

> 🤢 var names with only one letter!

It's important to use appropriate language when writing code reviews and to avoid using ALL CAPS, or insults, or any kind of unmannered expressions. This not only shows respect for your colleagues, but it also prevents any potential misunderstandings or conflicts with clients who may also be reviewing the code.

More "do not" example on code reviews:

> ARROW ANTIPATTERN, SERIOUSLY? 

Better do it this way:

> This code is becoming an arrow anti-pattern. Split the responsibility of each check and refactor this huge function.  Please check this article that explains why is it bad https://...

Polite language such as "may I suggest this?", "I tried this one and it performed faster", or "I would suggest this" is more appropriate when providing feedback. Additionally, instead of simply suggesting changes, it's a good practice to ask the developer who submitted the code for their reasoning, to understand their thought process while coding that branch. For example, "Why did you use let instead of const?"

By using polite language, and understanding the reasoning behind the code, it will help to create a positive and collaborative environment for the team and clients.

When providing feedback during code review, it's essential to be helpful and not assume that others already know what to do. Feel free to chat with your work colleague in Slack to get further  context. 

It's important to explain the reasoning behind your analysis, for example, "If you use the spread operator in this way, it will copy all key-value pairs at once, avoiding the need to update the code manually in the future if the main object is changed". Additionally, by providing context and the reasoning behind your suggestions, it helps the developer understand the benefits of the changes and how it will improve the code.

It's also helpful to provide relevant resources, such as links to other code reviews, articles, official documentation, etc. to support your suggestions. This will help the developer understand the reasoning behind your feedback and also give them more information to understand the context of the code changes.

In summary, always try to be helpful and informative when providing feedback during code review. Explain the rationale behind your suggestions and provide resources to support your feedback. This will help the developer understand the changes and the benefits it will bring to the codebase.

The result of a code review should be a "+1" comment in the PR stating you're OK with that code.

### Merging pull requests

Please follow this procedure:

* Add the "[RFC] …title of the PR here".
* Paste the PR url on the slack channel, and ask for code review
* Tag @name any developer or team lead in your PR description asking for advice or review as a recommendation.
* Don't ask for +1 comments.
* Wait at least 1 day if no "+1" comments or review appear, merge the branch.
* Give time to people to review, some of them are overseas or in Asia and it will take time to review.
* Owners must wait at least two "+1" comments in the PR to merge branches.

If the branch is not a feature, but a [BUG] or [HOTFIX], the idea is to code review it, but since it's critical procedure, proceed to merge it and pass all tests.

If the branch is [DOC] or [PERFORMANCE] just merge it right away, don't wait for approvals unless you specifically want to be reviewed.

### Using software

Although the use of Git UI applications is becoming popular, using git from Terminal is a really important skill. Please read this interesting tutorial from GitLab, and it’s really complete https://docs.gitlab.com/ee/topics/git/