---
title: "[Windows] Building A Blog with Github & jekyll"
date: 2025-08-11 18:30:00 +09:00
last_modified_at: 2025-08-12 10:48:00, +09:00
categories: [ETC., Building My Own Blog]
tags: [Github, Github Action, Jekyll, Ruby, Blog, Chirpy]     # TAG names should always be lowercase
description: Common Challenges When Building a Jekyll Blog on Windows
---

## I Built My Blog!
I have finally succeeded in building my own blog using GitHub and Jekyll.

After completing this process, I realized that the most important first step is to search for and follow the official documentation. It took me about three hours to configure the basic environment with Ruby and the Jekyll Chirpy theme. I believe I could have finished much faster if I had consulted the official guide instead of following a blog post that was two years old.

Regardless, I'm pleased that I was able to accomplish this on my own.

Here are some of the issues I encountered during the setup process.

## The ridk Command Wasn't Working in VS Code PowerShell

The following command did not work in the VS Code PowerShell terminal: 

```powershell
ridk version
```

I first tried to solve this with ChatGPT, but simple Googling provided the answer.

This issue was caused by the Windows PowerShell execution policy. Therefore, it was necessary to change this setting by following these steps:

1. Run PowerShell as an administrator.

2. Check your current policy with the command: `Get-ExecutionPolicy`

3. If the status is Restricted (or anything other than RemoteSigned), you need to change it with: `Set-ExecutionPolicy RemoteSigned`

After this, the command worked properly in the VS Code terminal.

## An Issue with Nokogiri Installation
While running `bundle install`, I encountered this error:

```powershell
An error occurred while installing nokogiri (1.18.9), and Bundler cannot
continue.
```
- The solution was to completely uninstall Ruby from the Windows Control Panel.

- If the uninstallation doesn't resolve the issue, check your system's environment variables and remove any paths that contain 'Ruby'.

## Changes to `_config.yml` Weren't Appearing
- This was not an issue with GitHub Actions.

- I learned that it simply takes some time for the server to update with the changes, even after the GitHub Action deployment is complete.


## How to Change the Author Name & A Post Wasn't showing up
- To change the author, you only need to edit the username in the _config.yml file. It's not necessary to add author information to the top of each Markdown file.

- For posts to appear, the filename must follow the Chirpy theme's convention: YYYY-MM-DD-title.md.

These were the four main challenges that stood out to me. I created this blog to document my 'Today I Learned' (TIL) journey, with a special focus on LLMs. I plan to start by summarizing some landmark papers in the field.

> ***## Additional Tips***
> - It is better to check if your changes are working by running a local server rather than repeatedly committing and pushing changes.
> - If `jekyll serve` doesn't work, try `bundle exec jekyll serve`.
> - To change the social media links below your profile, you need to edit the `_data/contacts.yml file`.
> - To display the post's "last updated" date, add `last_modified_at` to the front matter.