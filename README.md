# composer-experiment

A package that claims to be PHP.

```json
"type": "metapackage",
"provide": { "php": "8.5.9999" }
```

That is the whole package. It installs no files — a metapackage cannot — and
the version it claims is one nobody will ever ship, so anything it satisfies,
it satisfies visibly.

## What it is for

Composer treats `php` and `ext-*` as *platform* packages: they are not
installed, they are facts about the machine, discovered by asking the runtime.
A package may nonetheless declare `provide: {"php": ...}`, and the question
this repository exists to answer is what the ecosystem then does about it.

Two halves, and they are answered in different places:

**The resolver.** Already answered, and the answer is yes: with
`config.platform.php` pinned to `7.0.0`, a package requiring `php >=8.1`
resolves cleanly once this metapackage is in `require`, and fails without it.
That is not a loophole — it is the same mechanism that lets
`symfony/polyfill-mbstring` satisfy `ext-mbstring`.

**The registry.** Open. Packagist indexes providers of a name and serves them
at `providers/<name>.json`, which is how a tool can answer "nothing here gives
you `ext-iconv`, but these packages claim to". Today:

```console
$ curl -s https://packagist.org/providers/php.json
{"providers":[]}
```

Empty — but composer's own schema validation has no objection to the manifest
above, so the emptiness looks like *nobody has done it* rather than *it is not
allowed*. Publishing this package settles which.

## Why the answer matters

A language that wants to run PHP's ecosystem needs a way to say "the thing
your package requires is here, supplied differently". Doing that through
`provide` means using a field composer already understands, on a registry that
already exists, with no fork of either — a package manager on the other side
sees an ordinary dependency and installs it.

If the registry refuses the label, that route closes and compatibility has to
be declared somewhere the registry does not read, which every existing tool
would then have to be taught about.
