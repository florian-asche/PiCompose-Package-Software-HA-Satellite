# What does this implement/fix?

<!-- Quick description and explanation of changes. -->

**Related issue (if applicable):**

- Fixes #<issue number>

## Version bump

<!--
Tick exactly one box. It decides how the version of the package is raised when this PR is merged.
CI sets the label major, minor or patch from it. The check fails until there is exactly one.
-->

- [ ] Incompatible change, for example a renamed package or a changed config path — `major`
- [ ] New feature or new functionality — `minor`
- [ ] Fix or small change — `patch`

## Types of changes

<!--
Tick exactly one box. CI derives the label from the ticked box.
-->

- [ ] Bugfix (non-breaking change which fixes an issue) — `bugfix`
- [ ] New feature (non-breaking change which adds functionality) — `new-feature`
- [ ] Enhancement to an existing feature — `enhancement`
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected) — `breaking-change`
- [ ] Refactor (no behaviour change) — `refactor`
- [ ] Documentation only — `documentation`
- [ ] Maintenance / chore — `maintenance`
- [ ] CI / workflow change — `ci`
- [ ] Dependencies bump — `dependencies`

## Tested on

<!--
Packages that install drivers or services cannot be fully tested without hardware. Say what you
actually ran, for example the hardware and package version, or "built and linted only".
-->

- Hardware / package version:
- Result:

## Checklist

- [ ] The package builds with `dpkg-buildpackage -us -uc -b` and passes lintian.
- [ ] The title of this PR is fit to be a changelog entry, it becomes one line of the package changelog.
- [ ] `README.md` and `docs/` match the change.
- [ ] Breaking changes (renamed packages, changed config paths, changed dependencies) are described in the PR.
- [ ] I reviewed and understand every change in this PR (see [AI policy](AI_POLICY.md)).
