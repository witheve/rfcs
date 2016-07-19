<p align="center">
  <img src="https://github.com/witheve/assets/blob/master/images/logo.png?raw=true" alt="Eve logo" width="10%" />
</p>

# Eve Requests for Comments (RFC)

RFCs are meant to be an informal communication for starting a discussion on a particular feature, design, protocol, process, or anything else relating to Eve and the Eve community.

### Under Development

Our community is still in the early stages of development, so we're still figuring out how we want this process to work. Most likely, the RFC process will continually evolve to accommodate the needs of the Eve community as it grows.

# When to write an RFC

There is no one "test" that elevates a pull request to an RFC. However, one good rules is that an RFC is appropriate when the proposal affects disparate areas of the Eve community, especially across team boundaries. For example, a change to the syntax would affect everything from the parser to user programs, so it would certainly warrant an RFC.

By contrast, an RFC would not be appropriate for a small bugfix, or largely cosmetic changes.

# The RFC Process

We're trying to keep this process pretty informal to encourage as much participation as possible, but we also want to have some mechanisms in place to keep things organized and running smoothly. Here is an overview of how the RFC process it works:

1. A community member submits an RFC in the form of a pull request to this repository. Proposed RFCs should be placed in the "proposed" folder.
2. The Eve community will review and categorize the RFC. As long as the pull request is not a duplicate, it will be accepted and merged into this repository, and an issue will be opened to discuss the RFC.
3. The relevant stakeholders concerning the RFC will be identified and invited to discuss the RFC.
3. All stakeholders will discuss the RFC, and attempt to build consensus and integrate changes into the RFC.
4. Once consensus is reached, the RFC will either be accepted or rejected. An accepted RFC will be moved into the "accepted" folder of this repository. A rejected RFC will be removed from the "proposed" folder and its corresponding issue will be marked as closed.

An accepted RFC means that the proposal is on its way to becoming part of Eve, but the serious work of implementing the feature is still ahead.

# How to write an RFC

There is no particular format or length for an RFC, but the following section headers are a good place to start:

1. Summary
2. Motivation
3. Design
4. Implementation
5. Drawbacks
6. Alternatives
7. Risks

Again, feel free to use any or all of these sections, or add your own as you see fit.
