---
layout: post
title: "Tired of Forgetting Bash Commands? I Built a Rust TUI to Help!"
description: "I often find myself in a situation where I need to execute a command I know I've run before, but… what was that exact syntax again?"
date: 2025-02-09 19:24:00 +0100
category: bash-commander
tags: linux bash-commander
image: /assets/2025-02-09-bash-commander/screenshot.png
---

![Screenshot of bash-commander main interface](/assets/2025-02-09-bash-commander/screenshot.png)

As a developer managing a few Linux servers, including my own home server, I'm constantly SSH-ing in to perform various tasks. From system updates and service restarts to file management and monitoring, the command line is my daily bread and butter.

However, like many of you, I often find myself in a situation where I need to execute a command I know I've run before, but… what _was_ that exact syntax again? Was it `systemctl restart myservice` or `sudo systemctl restart myservice`? Did I need that extra flag for `rsync` to preserve permissions?

I experimented with creating aliases. While aliases offered a way to shorten frequently used commands, maintaining a growing list of them became cumbersome, and I still had to remember each alias name – even if it was short, it was yet another thing to memorize.

For a while, my solution was a collection of bash scripts. I'd keep a folder full of `.sh` files, each containing a command or a series of commands I wanted to remember and reuse. It worked, _sort of_. But honestly, scrolling through a directory to find the right script, then executing it, wasn't exactly practical. It felt clunky and inefficient.

I even dabbled with `whiptail`. It offered a nice-looking text-based menu, and I thought it could be the perfect solution. While visually appealing in the terminal, adding or editing entries was not user-friendly, and annotating commands with additional helpful information was just not really possible. Plus, `whiptail` isn't universally available on all platforms I use.

## Enter: Bash Command Saver - My Rust Experiment

Frustrated with these limitations and wanting a more elegant solution, I decided to tackle this problem myself. And since I've been eager to dive into Rust for a while, I figured this would be a perfect, practical project to learn the ropes. And so, "bash-commander" (working title!) was born.

Now, let's be clear: this is very much a **first working version**. It’s far from feature-complete, and I’m sure there are bugs lurking. But the exciting part is – **I'm already using it every day**, and it’s already making my server management workflow smoother.

### So, what does Bash Command Saver do (in this early version)?

Here’s a quick rundown of the current functionality:

- **Save Commands with Details:** You can add new bash commands and give them a descriptive name, a more detailed description, the actual command string itself, and even categorize them. This helps in organizing and remembering what each command does.
- **Edit and Delete Commands:** Need to tweak a saved command? No problem. You can easily edit existing commands or delete the ones you no longer need directly within the TUI.
- **Filter as You Type:** Got a growing list of commands? The built-in filter lets you quickly narrow down the list by typing keywords. It searches across names, descriptions, commands, and categories, making it fast to find what you're looking for.
- **Execute on Exit:** The core idea is simplicity. You select a command from the list using the arrow keys, and hit `Enter` to mark it for execution. Then, when you exit the application (using `Ctrl+q`), **the selected command, and only then, gets executed in your terminal.** This keeps the TUI focused on management and avoids accidental command execution within the interface itself.
- **Persistence is Key:** All your saved commands are stored in a file. When you start the application, it automatically loads your command catalog, ensuring your valuable command snippets are always readily available across sessions.

### How Does It Work Under the Hood? (Simplified Explanation)

The app is built using Rust and the fantastic Ratatui crate for creating the terminal user interface. Here's a simplified overview of the workflow:

1.  **Loading Commands:** On startup, the application reads commands from a designated file (currently stored in your user's home directory). These commands are parsed and stored in memory.
2.  **TUI Interaction:** The application presents a TUI in your terminal. You can navigate the list of commands, filter them, add new ones, edit existing ones, and delete commands through intuitive keyboard shortcuts.
3.  **Saving Selection:** When you select a command and press `Enter`, the application ends and executes it immediately. The output of the command is then displayed in your terminal after the TUI closes. To execute command i've used `std::os::unix::process::CommandExt` so it probably won't work on windows (sorry).
4.  **Saving on Exit:** Before exiting, the application saves any changes you've made to your command list back to the file, ensuring persistence for the next time you run the tool.

### What’s Next?

Bash Command Saver is still in its early stages. I have a long list of features I want to add and improvements to make. Things like:

- UI improvements
- Commands categorisation
- Import
- Cross-Platform Compatibility and distribution in packages (like \*.deb for debian)

And much more!

### Give it a Try

If you are also tired of command-line amnesia and are looking for a simple TUI-based command manager, give Bash Command Saver a try! Keep in mind it's a work in progress, but feedback and suggestions are very welcome as I continue to develop it.

[bash-commander repository on github](https://github.com/macap/bash-commander)
