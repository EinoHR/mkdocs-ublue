# Contributing
Thanks for taking the time to look into helping out!
All contributions are appreciated! 
Please refer to our [Code of Conduct](/CODE_OF_CONDUCT) while you're at it!

Feel free to report issues as you find them, and [helping others in the discussions]() is always appreciated.

## Getting Started

All types of contributions are encouraged and valued.
Please make sure to read the relevant section before making your contribution.
It will make it a lot easier for us maintainers and smooth out the experience for all involved.
The community looks forward to your contributions!

If you like the project, but just don't have time to contribute, that's fine. There are other easy ways to support the project and show your appreciation, which we would also be very happy about:

  - Star the project
  - Toot about it
  - Refer this project in your project's readme
  - Mention the project at local meetups and tell your friends/colleagues

## Code of Conduct

This project and everyone participating in it is governed by the
[CONTRIBUTING.md Code of Conduct](/CODE_OF_CONDUCT).
By participating, you are expected to uphold this code.
Please report unacceptable behavior to jorge.castro@gmail.com

## I Have a Question

If you want to ask a question, ask in the [discussion forum](https://github.com/orgs/ublue-os/discussions) or come hang out on [the Discord server](https://discord.gg/WEu6BdFEtp)

## I Want To Contribute

!!! warning "Legal Notice"
    When contributing to this project, you must agree that you have authored 100% of the content, that you have the necessary rights to the content and that the content you contribute may be provided under the project license.

Generally speaking we try to follow the [Lazy Concensus](http://lazyconcens.us/) model of development to keep the builds healthy and ourselves happy. 
    - If you're looking for concensus to make a decision post an issue for feedback and remember to account for timezones and weekends/holidays/work time 
    - We want people to be opinionated in their builds so we're more of a loose confederation of repos than a top-down org
    - Try not to merge your own stuff, ask for a review. At some point when we have enough reviewers we'll be turning on branch protection

## Reporting Bugs

### Before Submitting a Bug Report

A good bug report should describe the issue in detail. Generally speaking:

- Make sure that you are using the latest version
- Remember that these are unofficial builds, it's usually prudent to investigate an issue before reporting it here or in Fedora!
- Collect information about the bug:
    - `rpm-ostree status -v` usually helps
- Image and Version 
- Possibly your input and the output
- Can you reliably reproduce the issue? And can you also reproduce it with older versions?

### Why we insist on using GitHub

If you come to the Discord you might be asked to report an issue or start a discussion on GitHub. DON'T PANIC! 

We do this for a few reasons:

- Scalability - we're purposely designing this project to scale, and that means a focus on:
- Asynchronous Communication - OSS is a worldwide phenonmenon for a reason, forcing us to write everything down and capturing as much data is crucial to our ability to move quickly, at some point we'll have people in every time zone, and keeping that efficient is the key. 
    - Discord search is not an engineering tool, it's for chat, it's extremely difficuly for even the most experienced person to "spin up" by tracking a chat than an issue on GitHub.
    - It feels slow! It does, but over the long term, it is much more efficient, because you are also solving the problem for as many people as you can, so we gotta make it count!
- Leverage chat as much as you can to debug and move fast
    - But when things get over a few messages, start to copying stuff into a text editor so you have it
    - and then file an issue, you can always edit and fix it up later, concentrate on the capture.
    - It's hard to have that discipline, that's why we have teammates.
    - "Oh I'll just file it later" is a good way to learn something, twice. It's going to happen but knowing the trap is a good way to avoid it (or end up gravitating towards it!)
- **DON'T BE AFRAID OF FILING AN ISSUE**
    - Part of our culture is to _teach_ and help each other grow
    - Better to file an issue and have it closed than let a subtle problem spiral out of control
        - Having an issue closed means you've ruled something out, that can be just as important as a solution!
    - That also doesn't mean to file 47 issues around your favorite feature in a 10 minute period
        - If something needs more discussion, [file a proposal](https://github.com/orgs/ublue-os/discussions?discussions_q=is%3Aopen+label%3Aproposal)
    - You've earned it
        - The commons depends on everyone chipping in, be proud of your contribution!


### How to test incoming changes

One of the nice things about the image model is that we can generate an entire OS image for every change we want to commit, so this makes testing way easier than in the past.
You can rebase to it, see if it works, and then move back.
This also means we can increase the amount of testers! 

We strive towards a model where proposed changes are more thoroughly reviewed and tested by the community.
If you see a pull request that is opened up on an image you're following you can leave a review on how it's working for you.
At the bottom of every PR you'll see something like this: 

![image](https://user-images.githubusercontent.com/1264109/221305388-3860fc07-212c-4eb9-80d9-5d7a35a77f46.png)

Click on "Add your review", and then you'll see this: 

![image](https://user-images.githubusercontent.com/1264109/221307636-5e312e48-821f-4206-848f-7fbc2c91cd78.png)

Don't worry, you can't mess anything up, all the merging and stuff will be done by the maintainer, what this does is lets us gather information in a more formal manner than just shoving everything in a forum thread.
The more people are reviewing and testing images, the better off we'll be, especially for images that are new.

At some point we'll have a bot that will leave you instructions on how to rebase to the image and all that stuff, but in the meantime we'll leave instructions manually. 

Here's an example: https://github.com/ublue-os/nvidia/pull/49

## Building Locally

The minimum tools required are `git` and a working machine with `podman` enabled and configured.
Building locally is much faster than building in GitHub and is a good way to move fast before pushing to a remote.

### Prerequisites

#### On the developer host

- Add and entry for the container registry to your `/etc/hosts` file

   ```ini
   # Container registry
   127.0.0.1 registry.dev.local
   ```  

- Start the container registry

   `podman run -d --name registry.dev.local -p 5000:5000 docker.io/library/registry:latest`

#### On the VM where you want to test the image

- Find out the ip address for the default gateway

  `ip route | grep "default via" | cut -d ' ' -f 3` 

  _Example:_
  
  ```sh
  $ ip route | grep "default via" | cut -d ' ' -f 3
  10.0.2.2
  ```

- The VM needs to find the container registry on the host system. Add the ip address of the default gateway to the `/etc/hosts` file

   ```ini
   # Container registry
   10.0.2.2 registry.dev.local
   ```

- Since the container registry is running in "insecure" mode we have to create the file `/etc/containers/registries.conf.d/ublue-dev.conf`
   with the following configuration.

   ```toml
   [[registry]]
   location = "registry.dev.local:5000"
   insecure = true
   ```

### Clone the repository

Clone the repository of your choice. In this example we are using <https://github.com/ublue-os/main>

`git clone https://github.com/ublue-os/main.git`

### Build the image

- Make sure you can build the existing image

   `podman build . -t registry.dev.local:5000/ublue-main:dev-latest`

- Upload image to the container registry

   `podman push --tls-verify=false registry.dev.local:5000/ublue-main:dev-latest`

### Rebase to DEV image

To rebase your test VM to the DEV image you can run

`rpm-ostree rebase ostree-unverified-registry:registry.dev.local:5000/ublue-main:dev-latest`

### Make your changes

This usually involved editing the `Containerfile`.
Most techniques for building containers apply here, if you're new to containers using the term "Dockerfile" in your searches usually shows more results when you're searching for information. 

Check out CoreOS's [layering examples](https://github.com/coreos/layering-examples) for more information on customizing. 

## Reporting problems to Fedora

We endevaour to be a good partner for Fedora.

This project is consuming new features in Fedora and ostree, it is not uncommon to find an issue.
It is strongly recommended that you attempt to reproduce the issue with a vanilla Fedora installation to see if it's a Universal Blue issue or a Fedora issue. 
If you are confused and can't confirm, ask in chat or post in the forums and see if someone more experienced can help out.
Issues that you can confirm should be reported upstream, and in some cases we can help test and find fixes.
Some of the issues you find may involve other dependencies in other projects, in those cases the Fedora team will tell you where to report the issue. 

Upstream bug tracker: [https://github.com/fedora-silverblue/issue-tracker/issues](https://github.com/fedora-silverblue/issue-tracker/issues)

## Styleguides
### Commit Messages

We use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) and enforce them with a bot to keep the changelogs tidy:

```
chore: add Oyster build script
docs: explain hat wobble
feat: add beta sequence
fix: remove broken confirmation message
refactor: share logic between 4d3d3d3 and flarhgunnstow
style: convert tabs to spaces
test: ensure Tayne retains clothing
```

## Making a Release

Releases are automated via [Release Please](https://github.com/googleapis/release-please) with additional modifications to publish images. Since the ISOs are netinstalls and always pull the latest image you usually don't need to do a release unless new ISOs are needed or for human reasons like incrementing a version number.

1. Release please always opens a draft PR in https://github.com/ublue-os/main that tracks changes from the last release
1. Approving then merging the open PR will kick off the release action
    - If there is no open PR commiting anything to the repo will force the action to open a PR
    - `chore:` is ignored, so it must be something else, `fix:` or `docs:` is recommended 
1. The release action will then make a release
    - There is a delay as the ISOs need to be built, they will get attached to the same release after, this can take as long as 10 minutes.
    - Do not touch `CHANGELOG.md`, the action handles that.
    - It might be prudent to edit the release directly after to add topical links (website, gotchas) since we don't have a release template, we should make one.

## Join The Project Team

If you're interested in _maintaining_ something then let us know!

## Attribution
This guide is based on the **contributing.md**. [Make your own](https://contributing.md/)!
