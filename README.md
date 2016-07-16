<p align="center">
  <img src="http://www.witheve.com/logo.png" alt="Eve logo" width="10%" />
</p>

# Eve Requests for Comments (RFC)

RFCs are meant to be an informal communication for starting a discussion on a particular feature, design, protocol, process, or anything else relating to Eve and the Eve community.

## Under Development

Our community is in the early stages, so we're still figuring out how we want this process to work. Most likely, the RFC process will continually evolve to accommodate the needs of the Eve community as it grows.

# When to write an RFC

RFCs are meant to facilitate discussion on issues that are larger or more controversial than would be appropriate for typical pull request. For example, this could include large design changes, changes to internal architecture, new feature proposals, etc. An RFC would not be appropriate for a small bugfix, or largely cosmetic changes.
There is no one "test" that elevates a pull request to an RFC. However, one good rules is that an RFC is appropriate when the proposal affects disparate areas of the Eve community, especially across team boundaries. For example, a change to the syntax would affect everything from the parser to user programs, so it would certainly warrant an RFC.

# The RFC Process

We're trying to keep this process pretty informal to encourage as much participation as possible, but we also want to have some mechanisms in place to keep things organized and running smoothly. Here is an overview of how the RFC process it works:

1. A community member submits an RFC in the form of a pull request to this repository.
2. The Eve community will review and categorize the RFC.
3. The relevant stakeholders concerning the RFC will be identified and invited to discuss the RFC.
3. All stakeholders will discuss the RFC, and attempt to build consensus and integrate changes into the RFC.
4. Once consensus is reached, the RFC will either be accepted and merged into this repository, or rejected.

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
8. 
Again, feel free to use any or all of these sections, or add your own as you see fit.

# Opening an RFC

To open an RFC, just draft your proposal and create a new pull request in this repository. Your RFC will be tagged and reviewed by someone in the Eve community (we still need to work out an infrastructure for this process), and the RFC will be discussed by the community. They may suggest changes, point out competing or similar RFCs, and eventually the RFC will either be merged or rejected.
If the RFC is merged, it will be assigned an issue where discussion can take place. Depending on the RFC, discussion may be directed to an external venue, but for transparency and posterity, we would appreciate that any discussion in an alternative venue be logged (meeting minutes, summary of a conversation, chat transcript, etc.) and linked in the original issue.
