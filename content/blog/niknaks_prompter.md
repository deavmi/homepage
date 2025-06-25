---
title: "Addition to Niknaks: Terminal-based prompter mechanism"
author: Tristan B. Velloza Kildaire
date: 2025-03-01
draft: true
---

## Prompter and prompts

When working on the Tristan's Programming Language's package manager `tpkg` I
realised the quick need for a mechanism that would let me build up custom
questions and then have the answers be provided by the means of some `File`.

I wanted to basically be able to prompt the user, on the command-line, for
answers to several questions such as _"What is the name of the new module?"_
or _"What description do you want to have set?"_ and then have it do the
prompting for me and storing of the respective answers.

Well, that's what this basically does.

### Example code

A nice example of the prompting is a snippet of code out of the `tpkg`
code base itself. Here I prompt for a few things, notably I require the
answers for the prompts to the _package name_ and  _package description_
to be non-empty (that is the second `false` given to the `Prompt` constructor;
the first `false` means to not request a multiple-value answer).

```d
Prompter prompter = new Prompter(stdin);
prompter.addPrompt(Prompt("Project name: ", false, false));
prompter.addPrompt(Prompt("Project description: ", false, false));


Prompt[] answers = prompter.prompt();

Project proj;
string projName;
if(answers[0].getValue(projName))
{
    proj.setName(projName);
}
else
{
    ERROR("Could not get a valid project name");
    return;
}
string projDescription;
if(answers[1].getValue(projDescription))
{
    proj.setDescription(projDescription);
}
else
{
    ERROR("Could not get a valid project description");
    return;
}
```