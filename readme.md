# Introduction

### What is gradle/gradlew?
[Gradle](https://docs.gradle.org/current/userguide/userguide.html) is a build tool. It can be run with a [wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html), `gradlew`. The wrapper:
> invokes a declared version of Gradle, downloading it beforehand if necessary

This is great for ensuring developers and CI are using the same version of Gradle.

### What are some problems with the gradle wrapper?
There a few main problems regarding *security*:
1. Malicious `gradle-wrapper.jar` files may be added to PRs. It's hard to know if these are trustworthy - there's no easy-to-read diff. The Gradle team made a [wrapper-validation-action](https://github.com/gradle/wrapper-validation-action) to validate checksums in PRs. Whilst this does mitigate the problem, not every project is using this.
2. The aforementioned validation action may not be up-to-date, so there could be small windows of vulnerability. Tools like Dependabot can make this less likely - it can even auto-update [Github Actions](https://github.com/gradle/wrapper-validation-action/pull/58).
3. Dependabot can update dependencies & plugins in the gradle ecosystem - but **Dependabot doesn't currently update the gradle wrapper (see[1 - dependabot-core](https://github.com/dependabot/dependabot-core/issues/2223)/[2 - gradle](https://github.com/gradle/gradle/issues/20438)).**

Whilst you can manually update the gradle wrapper .jar/properties/scripts with `./gradlew wrapper --gradle-version ${VERSION} --distribution-type all`, it's not automatic.
It would be nice to remove as much toil as possible around dependency/tooling upgrades.

Whilst there are Github Actions like [update-gradle-wrapper-action](https://github.com/marketplace/actions/update-gradle-wrapper-action), many users already on dependabot may not hear about this action or go to find it. If Dependabot could update the gradle wrapper, it would make things [safer and faster](https://docs.gradle.org/7.0.2/release-notes.html#performance-improvements) for a huge number of users.

### This repo can be used to test dependabot-core

The `build.gradle` has two outdated dependencies:

1. Plugin: 'org.springframework.boot' version '2.6.0' (at least 2.7.1 is available)
2. Dependency: 'org.junit.jupiter:junit-jupiter-api:5.0.0' (at least 5.8.2 is available)

In addition, the gradle wrapper is outdated:

3. `./gradlew wrapper --gradle-version 7.4.1 --distribution-type all` was used (at least 7.4.2 is available)

### Testing dependabot-core with this repo

This repo can be used with Dependabot's [_dry run_ mode](https://github.com/dependabot/dependabot-core#dry-run-script).
*Currently*, the dependabot-core dry only picks up on dependencies 1 and 2:

```shell
bin/dry-run.rb gradle IdiosApps/dependabot-gradlewrapper-test
...
=> updating 2 dependencies: org.junit.jupiter:junit-jupiter-api, org.springframework.boot
```

You can see actual [PRs for these on this repo](https://github.com/IdiosApps/dependabot-gradlewrapper-test/pulls). They were made by the [dependabot action](https://github.com/IdiosApps/dependabot-gradlewrapper-test/blob/main/.github/dependabot.yml).

If a PR to dependabot-core can also update the gradle wrapper, there should also be at least one more detected change (1, 2, and also 3). Gradle wrapper updates could include .jar/props/scripts, so at least one 1 file.
