This repository is a minimal reproducible example of Pants not enforcing the `[run].omit=*test*` option in `.coveragerc` when `[coverage-py].global_report=true` is enabled in `pants.toml`.

Bootstrap Pants:

```
$ curl -L -o ./pants https://pantsbuild.github.io/setup/pants && \
chmod +x ./pants
```

Running test goal in Pants with `[test].use-coverage` option enabled, `[run].omit=*test*` enabled but `[coverage-py].global_report=true` **disabled** (as it is in this repo):

```
$ ./pants test ::
```

Results in non-test source files being reported on as so:

```
Name                                    Stmts   Miss  Cover
-----------------------------------------------------------
minimalcov/minimalcov/__init__.py           0      0   100%
minimalcov/minimalcov/src/__init__.py       0      0   100%
minimalcov/minimalcov/src/foo.py            1      0   100%
-----------------------------------------------------------
TOTAL                                       1      0   100%
```

Activating global reporting by uncommenting `[coverage-py].global_report=true` in `pants.toml` shows the issue: Pants should inherit configuration options for `coverage.py` from `.coveragerc` and also omit test files in the same way but this does not happen.

Running the test goal again with `[coverage-py].global_report=true`:
```
$ ./pants test ::
```
Results in:
```
Name                                      Stmts   Miss  Cover
-------------------------------------------------------------
minimalcov/minimalcov/__init__.py             0      0   100%
minimalcov/minimalcov/src/__init__.py         0      0   100%
minimalcov/minimalcov/src/foo.py              1      0   100%
minimalcov/minimalcov/tests/__init__.py       0      0   100%
minimalcov/minimalcov/tests/test_foo.py       3      3     0%
-------------------------------------------------------------
TOTAL                                         4      3    25%
```


This use-case is valuable for identifying target directories that have source files but have yet to be given test files.