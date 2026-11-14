# Direct documentation

Public documentation for the parts of [Direct](https://github.com/Cratis/DirectPublic) that people install, run
or configure themselves, starting with the Direct Agent Runner.

Direct itself is a private product. Only documentation for public pieces belongs in this repository.

## Layout

```text
Documentation/
  toc.yml            Table of contents of the whole site section
  index.md           Overview
  runner/            Direct Agent Runner: overview, get started, install, configure, troubleshooting
    images/          Screenshots used by the pages
  forgejo/           Connect Forgejo: Codeberg or your own server, sign-in methods, agent accounts, webhooks
```

Each folder has its own `toc.yml`, as in the other Cratis product repositories.

## How it is published

The central [Cratis documentation site](https://github.com/Cratis/Documentation) checks this repository out and
publishes the `Documentation` folder as the **Direct** section at [cratis.io/direct](https://www.cratis.io/direct/).
Changes merged to `main` appear after the next site build.

## Writing

- American English, second person, present tense.
- Every page has `title` and `description` front matter and is listed in a `toc.yml`.
- Screenshots go in the `images` folder next to the page that uses them. Use made-up names, organizations and
  addresses, never real ones.
- Do not describe or link private Direct internals.

## License

MIT. See [LICENSE](./LICENSE).
