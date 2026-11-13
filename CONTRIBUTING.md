# Contributing

Thanks for helping with xl-fiber.

## Workflow

- `main` is the only long-lived branch. Development happens on it, and every release tag is cut from it.
- Fork the repository and create a branch such as `fix/...` or `feature/...`.
- Open a pull request against `main`. A maintainer reviews it and merges it.

## Before opening a PR

- Run `xl test`. All tests must pass.
- Add a test in `test/*.test.xl` for any new behavior.
- Write modern, compact xlang: classes, closures, `:=` and `for … in`.
- Keep commit messages short and clear.

## Developer Certificate of Origin

Sign off every commit (`git commit -s`). By signing off you certify the [DCO](https://developercertificate.org/).

## License

Your contributions are licensed under the [Apache License 2.0](LICENSE).
