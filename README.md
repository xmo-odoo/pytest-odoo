# pytest-odoo

Allows running post-install odoo tests inside pytest, compatible with xdist.
Requires a database with all the modules to test installed as input.

On my machine (Ryzen 5 7530U, 6 cores 12 threads, base clock 2GHz max boost
4.5GHz):

- installing a database with everything other than l10n modules (and a few
  nonsense modules e.g. pos_blackbox_be, social_demo, test_translation) and
  running at_install tests takes ~35mn
- a first run at `-n6`[^parallelism] takes 2:40:00
- a second run for random failures takes 2:50[^random]

For a total of ~3:20:00 wallclock, running at 374% overall but the parallel
phase runs at about 435%, not including postgres (but I'm pretty sure including
chrome)

The entire test suite would run in about 10~12 hours (I get 44540 seconds of CPU
which is 12:22:34, but that includes sync overhead and DB duplication which )

# usage

- Install `pytest-odoo` in your environment, or add it to your pyproject's
  `dependency-groups.dev`.
- Run the test suite with the `--db` parameter to specify the database to
  run post-install tests for.
- `--addons-path` can be used for addons path setups, but I don't use that
  so I would recommend a PYTHONPATH setup instead.

# pytest configuration

Setting a few options in `pytest.ini` is probably a good idea:
- `collect_imported_tests = false`, this is the behaviour of the builtin
  test runner, running imported test cases just wastes time as they're not
  intended for that
- `testpaths` listing the addons directories, so pytest's test discovery
  does a bit less trawling
- `filterwarnings` set up to ignore DeprecationWarning, pytest warnings, and
  a few others, as they make pytest angry by default

# complementary extensions

- [pytest-xdist], allows parallelising tests
- [pytest-sugar], provides a nice UI than the default
- [pytest-runbot-autotags], automatically fetches and skips tests which are
  disabled on the runbot
- [pytest-xfailer], allows maintaining a data-driven list of broken tests
  (as some are impossible to run locally -- at least on my machine)
- [odoo-deps-backend], not really necessary but makes running Odoo via uv easy

[^random]: the amount and stability of failures varies by day & install,
    so it can take multiple runs and a fair bit more time to get all tests
    to succeed
[^parallelism]: it's my experience that leaving the HT threads for chrome and
    postgres is a good idea to avoid the machine getting completely bogged
    down and unusable, as with that it regularly peaks to a full load or near
    enough, without HT (e.g. apple silicon) setting a few real cores aside is
    probably sufficient, although things are probably more complicated if
    E-cores are a large part of the cores count

[pytest-xdist]: https://github.com/pytest-dev/pytest-xdist
[pytest-sugar]: https://github.com/teemu/pytest-sugar
[pytest-runbot-autotags]: https://github.com/xmo-odoo/pytest-runbot-autotags
[pytest-xfailer]: https://github.com/xmo-odoo/pytest-xfailer
[odoo-deps-backend]: https://github.com/xmo-odoo/odoo-deps-backend
