# Zig multiplatform

🏷️ Single-executable self-extracting multiplatform Zig releases \
🔀 Forked from [ziglang/releases-cutter](https://github.com/ziglang/release-cutter)

<table align=center><td>

```sh
wget https://github.com/jcbhmr/release-cutter/releases/download/0.13.0/zig-ape-0.13.0.zip
unzip zig-ape-0.13.0.zip
./zig-ape-0.13.0/zig zen
```

</table>

🌌 Uses [`cosmocc`](https://github.com/jart/cosmopolitan/tree/master/tool/cosmocc) to build a multiplatform binary \
📂 Extracts the platform-specific `zig` and `lib/*` files to a cache folder \
💻 Works on Windows, macOS, and Linux on both x86-64 & AArch64 CPUs \
**😎 It's just one binary!**

## Installation

![Windows](https://img.shields.io/static/v1?style=for-the-badge&message=Windows&color=0078D4&logo=Windows&logoColor=FFFFFF&label=)
![Linux](https://img.shields.io/static/v1?style=for-the-badge&message=Linux&color=222222&logo=Linux&logoColor=FCC624&label=)
![macOS](https://img.shields.io/static/v1?style=for-the-badge&message=macOS&color=000000&logo=macOS&logoColor=FFFFFF&label=)

**🛑 This is an unofficial redistribution of the Zig binaries.** You should probably use [the official Zig download page](https://ziglang.org/download/). 

1. Download the zip.
2. Extract it.
3. Rename it to `zig.com` or `zig.exe`. (Windows only)

**https://github.com/jcbhmr/release-cutter/releases/download/0.13.0/zig-ape-0.13.0.zip**

The all-in-one binary should work on the following platforms:

- Windows x86-64
- Windows AArch64
- macOS x86-64
- macOS AArch64
- Linux x86-64
- Linux AArch64
- FreeBSD x86-64

**For other platforms make sure you check out [Zig's official download page](https://ziglang.org/download/).**

## Development

![C++](https://img.shields.io/static/v1?style=for-the-badge&message=C%2B%2B&color=00599C&logo=C%2B%2B&logoColor=FFFFFF&label=)
![GitHub Actions](https://img.shields.io/static/v1?style=for-the-badge&message=GitHub+Actions&color=2088FF&logo=GitHub+Actions&logoColor=FFFFFF&label=)

This is a fork of Zig's release script repository. **This project's releases are not official.** I don't know if the Zig project wants or would want to publish a self-extracting self-executing multiplatform binary. 🤷‍♀️

Right now there's [a bug in Cosmopolitan Libc](https://github.com/jart/cosmopolitan/issues/1248) that has been fixed but hasn't been published as a release yet. You'll need to build `cosmocc` from source to use the fix.

The GitHub actions workflow doesn't actually build Zig from source; it pulls the precompiled artifacts from the [Zig downloads page](https://ziglang.org/download/) and moves them around to where the script expects the build artifacts to be. It also only runs the `./build-zig-ape` script and not the full `./script` release script.

TODO: [I need some assistance in handling non-zero exit codes and process signals properly in Windows.](https://github.com/jcbhmr/release-cutter/blob/4a39e0a5d1222105f544afa7ad33dfcb29181ac3/zig-ape.cpp#L106) If anyone reading this knows how to `waitpid()` correctly please enlighten me! ☺️

---

# Release Cutter

Helpful scripts for cutting a Zig release.

## Making a New CI Tarball

First, use [zig-bootstrap](https://github.com/ziglang/zig-bootstrap) to build the
relevant tarball(s) for the targets the CI needs.

Next, run the update-ci-tarballs.zig script. It has its own usage text to guide
you through the process. The final step is using `s3cmd` to upload the tarballs.
The script prints suggested commands for you to run which you can copy and paste.

Next you have to do CI chores to update to the new tarball. Each CI system has
their own different chore you have to do.

 * ci.ziglang.org
   - copy the x86_64-linux-musl tarball to ci.ziglang.org:/depot/deps
   - ssh into ci.ziglang.org and follow
     [these directions](https://github.com/ziglang/zig-ci/blob/main/docker/debian-amd64-11.1.md),
     with two modifications:
     - replace `zig+llvm+lld+clang-x86_64-linux-musl-0.9.1.tar.xz` and
       `zig+llvm+lld+clang-aarch64-macos-none-0.9.1.tar.xz` with your new tarballs.
     - bump the version of `debian-amd64:11.1-9` to the next number after the one in
       the main zig git repo, the `image` fields of `ci/zinc/drone.yml`.
   - in the main zig git repo, update the `image` fields of `ci/zinc/drone.yml`.
 * azure
   - in the main zig git repo, update `ZIG_LLVM_CLANG_LLD_NAME` in `azure/pipelines.yml`.
   - in the main zig git repo, update `CACHE_BASENAME` in `azure/macos_script`.
 * drone
   - rent an arm64 server from e.g. vultr or digitalocean
   - use [docker-zig](https://github.com/ziglang/docker-zig) to build and publish an updated
     docker image
   - in the main zig git repo, update the `image` fields of `ci/drone/drone.yml`
 * lavahut
   - in the main zig git repo, update `CACHE_BASENAME` in `srht/freebsd_script`.

### Template for Copying Artifacts from FreeBSD Server

1. Rent FreeBSD VPS and run zig-bootstrap on it.
2. cd into zig-bootstrap, run these with the IP edited:

```
rsync -avzu root@96.30.194.18:zig-bootstrap/out/x86_64-native-baseline/ ./out/x86_64-freebsd-gnu-baseline/
rsync -avzu root@96.30.194.18:zig-bootstrap/out/zig-x86_64-native-baseline/ ./out/zig-x86_64-freebsd-gnu-baseline/
```

Then you can use the update-ci-tarballs.zig script including for FreeBSD.

## GitHub GraphQL Snippets

### List All the Tiers

These are hard-coded into the script; if these change the .zig code must be updated.

```gql
{
  organization(login: "ziglang") {
    sponsorsListing {
      tiers(first: 20) {
        nodes {
          id
          name
          description
        }
      }
    }
  }
}
```

Output: (2021-12-18)

```
{
  "data": {
    "organization": {
      "sponsorsListing": {
        "tiers": {
          "nodes": [
            {
              "id": "MDEyOlNwb25zb3JzVGllcjM5NDY1",
              "name": "$5 a month",
              "description": "A meaningful contribution for a nimble organization. \r\n"
            },
            {
              "id": "MDEyOlNwb25zb3JzVGllcjM2Mjgy",
              "name": "$10 a month",
              "description": "Let's reclaim control of Open Source software development!"
            },
            {
              "id": "MDEyOlNwb25zb3JzVGllcjM2Mjgz",
              "name": "$15 a month",
              "description": "Like the previous tier, but it seems the compiler decided to add some padding. Thank you very much!"
            },
            {
              "id": "MDEyOlNwb25zb3JzVGllcjM0MDcy",
              "name": "$25 a month",
              "description": "This funds 30 minutes of paid developer work per month. Brilliant!"
            },
            {
              "id": "MDEyOlNwb25zb3JzVGllcjM0MDcx",
              "name": "$50 a month",
              "description": "This funds one hour of paid developer work per month.\r\n\r\nAs a reward:\r\n\r\n* Your name listed in the official release notes with every release of Zig."
            },
            {
              "id": "MDEyOlNwb25zb3JzVGllcjM0MDcw",
              "name": "$100 a month",
              "description": "This funds two hours of paid developer work per month. With just 40 other people donating this amount, it would be enough to hire someone.\r\n\r\nAs a reward:\r\n\r\n* Your name **with a hyperlink to your home page** listed in the official release notes with every release of Zig."
            },
            {
              "id": "MDEyOlNwb25zb3JzVGllcjM0MDc1",
              "name": "$200 a month",
              "description": "This funds 4 hours of paid developer work per month. With just 20 other people donating this amount, it would be enough to hire someone.\r\n\r\nAs a reward:\r\n * Your name **with a hyperlink to your home page** listed in the official release notes with every release of Zig.\r\n * Your name listed at the bottom of ziglang.org landing page."
            },
            {
              "id": "MDEyOlNwb25zb3JzVGllcjM0MDc3",
              "name": "$400 a month",
              "description": "This funds 8 hours of paid developer work per month. With just 10 other people donating this amount, it would be enough to hire someone.\r\n\r\nAs a reward:\r\n\r\n * Your name **with a hyperlink to your home page** listed in the official release notes with every release of Zig.\r\n * Your name **with a hyperlink to your home page** listed at the bottom of ziglang.org landing page.\r\n"
            },
            {
              "id": "MDEyOlNwb25zb3JzVGllcjUwMDE0",
              "name": "$1,200 a month",
              "description": "This funds 24 hours of paid developer work per month.\r\n\r\nA good option for companies that intend to donate using GitHub Sponsors.\r\nPlease contact us for alternative donation methods and any other requests.\r\n"
            },
            {
              "id": "MDEyOlNwb25zb3JzVGllcjUwMjgx",
              "name": "$5,000 a month",
              "description": "This funds 100 hours of paid developer work per month!\r\n\r\nWith your help, we're going to take off every Zig at the speed of light!\r\n"
            }
          ]
        }
      }
    }
  }
}
```

### Obtain the Full Data Set

Ideally the tool would do this automatically but I didn't bother to set that up
yet, and I'm not willing to do it in any other language than Zig. Pull requests
using other programming languages will be REJECTED!!!

In the meantime, there's this
[explorer thing](https://docs.github.com/en/graphql/overview/explorer).

```gql
query GetSponsors($tierId: ID) {
  organization(login: "ziglang") {
    sponsors(tierId: $tierId, first: 100) {
      nodes {
        ... on User {
          id
          name
          login
          twitterUsername
          websiteUrl
        }
        ... on Organization {
          id
          name
          login
          twitterUsername
          websiteUrl
        }
      }
    }
  }
}
```

* Do the query multiple times, annotating each result with its `tierId`
  (I couldn't figure out how to do this directly with GraphQL. Annotate
  each one by saving each query into a file named `tiers-data/$tierId.json`.
  - `{"tierId": "ST_kwHOAarWdc3EaQ"}`
  - `{"tierId": "ST_kwHOAarWdc3DXg"}`
  - `{"tierId": "ST_kwHOAarWdc2FHQ"}`
  - `{"tierId": "ST_kwHOAarWdc2FGw"}`
  - `{"tierId": "ST_kwHOAarWdc2FFg"}`
  - `{"tierId": "ST_kwHOAarWdc2FFw"}`

### Command Line Application

Now run the `sponsors-html` tool with no arguments and follow the help
instructions that it prints.

