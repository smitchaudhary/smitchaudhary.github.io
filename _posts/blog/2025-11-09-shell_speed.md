---
layout: single
title: "You should speed up your shell"
excerpt: "How long does it take your shell to start?"
date: 2025-11-09 21:25:34 +0100
categories: blog
permalink: /blog/shell-speed/
classes: wide
comments: false
author_profile: false
share: false
---

Imagine this: You have shared your screen while on a huddle with a colleague. They are helping you debug an issue. They ask you to spawn a new shell and run `pnpm dev`. Then "Can you do `uv run` in that other repo?". "Oh! In a new shell, can you launch another server?". And now imagine every single time you launch a new session, it takes about a second for the shell to start. That is a whole 1000ms or more for the shell to start. Every. Single. Time. Well, I don't have to imagine any of this, because unfortuantely for me, this is exactly what happened.

The first course of action of course is to measure this delay. You can time your shell start[^1] time using
```bash
for i in $(seq 1 10); do /usr/bin/time $SHELL -i -c exit; done
```
This gave me:
```
1.29 real         0.61 user         0.63 sys
1.21 real         0.60 user         0.65 sys
1.22 real         0.60 user         0.64 sys
1.22 real         0.60 user         0.65 sys
1.23 real         0.60 user         0.65 sys
1.12 real         0.60 user         0.65 sys
1.21 real         0.60 user         0.64 sys
1.21 real         0.60 user         0.65 sys
1.21 real         0.60 user         0.65 sys
1.21 real         0.60 user         0.65 sys
```

As you can clearly see, it is taking me somehwere between 1.2 to 1.3 seconds to start up a new shell. Now, that might seem like a small number, but it is far from that.

For one, our computers are very very fast. So blazingly fast, that 1300ms is practically forever. And it should not take this much to load my shell. All it has to do is load up my `zshenv`, `zprofile`, `zshrc`[^2] and so on. Which means, I am making it clunky, as we know the bare shell is near instantaneous. I really mean instantaneous. If I run `for i in $(seq 1 10); do /usr/bin/time bash -i -c exit; done` instead of the previous command, it tells me it took 0.00 seconds total. So that it the best case scenario.

Secondly, even if each individual prompt seems to take "only" 1.3 seconds, I launch dozens of shells each day. I pretty much live in the terminal. I love my GUI tools, but so much happens in the terminal these days. I open lots of throwaway Claude Code/Amp Code sessions for singular questions/investigations. Now I instinctively split the terminal pane, ask Claude/Amp where the logic for that part lies, and kill the pane as soon as I have the answer. So these slow times do add up.

And lastly, and arguably the most important is that I simply dislike it when things are slow and especially when I know it is because I am being wasteful. I know it is slow because it is bloated. And I know I am responsible for it. And I _don't_ want to be the kind of user who knowingly allows bloat. More so when I will be reminded of it multiple times a day.

## The Diagnosis

The next step is to actually find out what is taking up so much time. For that, we can use zsh's inbuilt profiler. We can do that by loading the profiler at the top of our `zshrc` file[^3]:
```bash
zmodload zsh/zprof
```
and follow it up by `zprof` at the very end of your `zshrc` to see which functions or calls are taking the most time.

With this in place, every time you source your zshrc file, it will print the results of the profiling it has done. You can do that a number of times and see which are the functions that end up taking a disproportianate amounts of time. The culprit(s) will reveal themselves as the ones that take up an order of magnitude more time than the rest.

I had some suspects in mind. Primary one being the prompt customization I have in place. I use [powerlevel10k](https://github.com/romkatv/powerlevel10k) and recently since I moved to using [jujutsu](https://jj-vcs.github.io/jj/latest/), I replaced the git VCS info from the right side prompt to jj info. But I had already taken care to optimize the prompt and strip it off the parts I do not need in addition to having instant prompt on for powerlevel10k, so now was the time to look at the numbers.

## The Fix

Turns out the real culprits were the parts that I did not really use and won't really miss if they are gone. One such thing was [NVM](https://github.com/nvm-sh/nvm), the Node Version Manager. It looked like it took up close to 700ms every time. Shaving that off was a no brainer. There were other smaller things that had crept their way into my config which I could clean. A bunch of googling, asking Claude and some surgeries later, now my shell was much faster and responsive.

Checking the shell start time again, now, we get the start up time to be ≈500ms. At this point, it is already much better than over a second that it was taking. A 2.5 times faster shell is a noticable difference, even if it is "merely" 700ms or so faster. But even at this stage, there is an obvious delay and it does not really feel instantaneous. I believe halving the shell start time again, without losing much of my beloved functionality and customizations would be ideal. So I went on to do a bit more of culling and cleaning. Saved some ≈70ms by getting rid of `rbenv`,  shaved of a few ms here and a few more there. Finally, I ended up with the following:

```
0.19 real         0.07 user         0.08 sys
0.14 real         0.06 user         0.06 sys
0.14 real         0.06 user         0.06 sys
0.14 real         0.06 user         0.06 sys
0.14 real         0.06 user         0.06 sys
0.14 real         0.06 user         0.06 sys
0.14 real         0.06 user         0.06 sys
0.14 real         0.06 user         0.06 sys
0.14 real         0.06 user         0.06 sys
0.14 real         0.06 user         0.06 sys
```

At this stage, the start up time is down to ≈140ms. This also passes the eye test, in the sense that the shell seems to start instantaneously to my perception.

## The Survivors

Profiling it again tells me there are still a few plugins and customizations I have that are responsible for majority of the time. The primary culprit in all this is the `fzf` history widget. It allows me to have fzf keybindings as well as the needed functions for fuzzy finding through previous commands using Cmd+R. And as stated earlier, for the amount of time I end up spending in the terminal, it adds enough value for me to take the slight performance hit.

Another such culprit is the zsh syntax highlighting plugin. Needless to say, this too, adds enough value to survive the great zshrc purge.

## The Conclusion

My earlier grandstanding about not wanting to be wasteful notwithstanding, I decided to stop with the optimization at this point. I am happy with the current state of my shell, and I believe that it is a good balance between performance and functionality. That being said, there are a few more things that can (and likely will) be done in the future. As always the biggest culprits will be on the chopping block first. That means, instead of loading the entire `fzf` plugin, I could only load the parts that I actually use. Similarly, as convenient as the `powerlevel10k` is, I could further optimize it (for example, currently, it loads up git info in all version controlled directories and overwrites it in `jj` directories).

If the shell seems to be getting any slower, I could always lazy load the things that I do not need every single time, but do need them in certain cases. This nearly 10 times faster shell start is a great start. Next up, is your turn, if you find yourself waiting for even just a bit after you open a new pane on tmux, remember that you are just a few profiling sessions away from a significantly faster experience. And if you find some interesting tricks, I would love to hear those.

## Update (21 Dec 2025)

Since this post, I continued optimizing my shell configuration. The startup time has now improved even further:

```
0.06 real         0.03 user         0.02 sys
0.04 real         0.02 user         0.01 sys
0.04 real         0.02 user         0.01 sys
0.04 real         0.02 user         0.01 sys
0.04 real         0.02 user         0.01 sys
0.04 real         0.02 user         0.01 sys
0.04 real         0.02 user         0.01 sys
0.04 real         0.02 user         0.01 sys
0.04 real         0.02 user         0.01 sys
0.04 real         0.02 user         0.01 sys
```

The shell now starts in approximately **40-60ms** on average. This represents another significant reduction from the ~140ms achieved at the time of writing. And orders of magnitude better than where we started. The optimization journey continues, but at this point, the startup time is imperceptible and the shell feels truly instantaneous. What is interesting is that I did not even need to sacrifice desired functinoality at the altar of optimization. Merely static loading some things instead of spawning shell processes and removing certain parts that had since become redundant was enough^[4].

---

[^1]: Well, tEchNiCaLly, it is the time to start and exit the shell, but of course the time to exit is negligible.

[^2]: I use zsh, the default on MacOS. But the same idea holds for other shells as well. It loads up the system wide configurations as well as user specific configurations in a predefined order.

[^3]: I already knew that the issue is with the `zshrc` file since other files like `zprofile`, `zshenv` and so on have been unchanged. And the `zshrc` file routinely gets polluted by programs wanting to add to path and so on.

[^4]: If you are interested, the file looks like [this](https://github.com/smitchaudhary/dotfiles/blob/2453d57fe91cb7c59343c1c5bc2414a52dc56c7a/zshrc) at the time of writing.
