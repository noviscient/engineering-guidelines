# Code Reviews Guidelines

Inspired by Sentry's [Code Review Guidelines](https://develop.sentry.dev/code-review/)

- [Identify problematic code](#identify-problematic-code)
- [Improve Design](#improve-design)
- [Tests Included](#test-included)
- [Assess and approve long-term impact](#assess-and-approve)
- [Double-check expected behavior](#double-check-expected)
- [Information sharing and professional development](#information-sharing)
- [Reducing code complexity](#reducing-code-complexity)
- [Guidelines for Submitters](#guidelines-for-submitters)
- [Guidelines for Reviewers](#guidelines-for-reviewers)

## <a name="identify-problematic-code">Identify problematic code</a>

Above all, a code review should try to identify potential bugs that could cause the application to break – either now, or in the future.

- Uncaught runtime exceptions (e.g. potential for an index to be out of bounds)
- Obvious performance bottlenecks (e.g. O(n^2) where n is unbounded)
- Code alters behavior elsewhere in an unanticipated way
- API changes are not backwards compatible (e.g. renaming or removing a key)
- Complex ORM interactions that may have unexpected query generation/performance
- Security vulnerabilities
- Missing or incorrect Permissions or Access Control.

## <a name="improve-design">Improve Design</a>

When reviewing code, consider if the interactions of the various pieces in the change make sense together. If you are familiar with the project, do the changes conflict with other requirements or goals? Could any of the methods being added be promoted to module level methods? Are methods being passed properties of objects when they could be passed the entire object?

## <a name="test-included">Tests Included</a>

Look for tests. There should be functional tests, integration tests or end-to-end tests covering the changes. If not, ask for them. At NoviTech we rely on our test suite to maintain a high quality bar and ship rapidly.

When reviewing tests double check that the tests cover the requirements of the project or that they cover the defect being fixed. Tests should avoid branching and looping as much as possible to prevent bugs in the test code from gaining a foothold.

Functional tests that simulate how a user would call our APIs or use our UX are key to preventing regressions and avoiding brittle tests that are coupled to the internals of the products we ship.

Tests are also the ideal place to ensure that the changes have considered permissions and access control logic.

## <a name="assess-and-approve">Assess and approve long-term impact</a>

If you’re making significant architectural, schema, or build changes that will have long-term ramifications to the software or data, it is necessary to solicit a senior engineer’s acknowledgment and blessing.

- Large refactors
- Database schema changes, like adding or removing columns and tables
- API changes (including JSON schema changes)
- Adopting new frameworks, libraries, or tools
- New product behavior that may permanently alter performance characteristics moving forward

## <a name="double-check-expected">Double-check expected behavior</a>

The reviewer should make a genuine attempt to double-check that the goals of the PR appear to be satisfied by the code submitted. This requires the submitter to write a good description of the expected behavior, and why. See also: Guidelines for submitters below.

## <a name="information-sharing">Information sharing and professional development</a>

Code reviews are an opportunity for more people to understand forthcoming code changes, so that they might in turn teach others down the road, and be in a position to fix something if/when the original author is not be available.

## <a name="reducing-code-complexity">Reducing code complexity</a>

Research shows that LOC is correlated with a higher bug count. If reviewers see an easy opportunity to significantly reduce the amount of code that is submitted, they should suggest a different approach.

## <a name="guidelines-for-submitters">Guidelines for Submitters</a>

### Try to organize your work in a way that makes it conducive to review

- Ideally, a pull request is limited to only a single feature or behavior change.
- This might feel like more work up-front, but it can make code review faster, reduce risk by letting you ship in stages, and ultimately end up being quicker.

### Describe what your PR does in a few sentences in the description field

- Additionally explain why we’re making these changes.
- If applicable, explain why other approaches were explored but not settled on.
- This gives the reviewer context, and prevents them going down the same rabbit holes that that submitter may have already explored when creating the code.

### Be your own first reviewer

- After you’ve put up your PR on GitHub, walk through the code yourself, before assigning an external reviewer.
- You’ll often catch code mistakes you didn’t see when writing it.
- This is also a good time to leave comments and refresh your memory in order to write a more helpful description.

### Avoid rebasing unnecessarily

- After a rebase, previous review comments will be orphaned from their now non-existent parent commits, making review more difficult
- Rewriting history makes it difficult for reviewers to isolate the scope of their review

## <a name="guidelines-for-reviewers">Guidelines for Reviewers</a>

### Be polite and empathetic

- Avoid accusatory and/or judgmental comments like: “You should have done X”

### Provide actionable feedback

- Instead of “This is bad”, try “I feel this could be clearer. What if you renamed variable X to Y?”

### Distinguish between “requires changes” and “nitpicks”

- Consider marking a PR as approved if the only requested changes are minor nits, so as not to block the author in another asynchronous review cycle.