
<!-- README.md is generated from README.Rmd. Please edit that file -->

# config <img src='man/figures/logo.svg' align="right" height="139" />

<!-- badges: start -->

[![CRAN
status](https://www.r-pkg.org/badges/version/config)](https://CRAN.R-project.org/package=config)
[![R-CMD-check](https://github.com/rstudio/config/workflows/R-CMD-check/badge.svg)](https://github.com/rstudio/config/actions)
[![Codecov test
coverage](https://codecov.io/gh/rstudio/config/branch/main/graph/badge.svg)](https://app.codecov.io/gh/rstudio/config?branch=main)
<!-- badges: end -->

The **config** package makes it easy to manage environment specific
configuration values. For example, you might want to use distinct values
for development, testing, and production environments.

You can install the **config** package from CRAN as follows:

``` r
install.packages("config")
```

## Usage

Configurations are defined using a [YAML](https://yaml.org/about.html)
text file and are read by default from a file named **config.yml** in
the current working directory (or parent directories if no config file
is found in the initially specified directory).

Configuration files include default values as well as values for
arbitrary other named configurations, for example:

**config.yml**

``` yaml
default:
  trials: 5
  dataset: "data-sampled.csv"
  
production:
  trials: 30
  dataset: "data.csv"
```

To read configuration values you call the `config::get` function, which
returns a list containing all of the values for the currently active
configuration:

``` r
config <- config::get()
config$trials
config$dataset
```

You can also read a single value from the configuration as follows:

``` r
config::get("trials")
config::get("dataset")
```

The `get` function takes an optional `config` argument which determines
which configuration to read values from (the “default” configuration is
used if none is specified).

## Configurations

You can specify which configuration is currently active by setting the
`R_CONFIG_ACTIVE` environment variable. The `R_CONFIG_ACTIVE` variable
is typically set within a site-wide `Renviron` or `Rprofile` (see [R
Startup](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Startup.html)
for details on these files).

``` r
# set the active configuration globally via Renviron.site or Rprofile.site
Sys.setenv(R_CONFIG_ACTIVE = "production")

# read configuration value (will return 30 from the "production" config)
config::get("trials")
```

You can check whether a particular configuration is active using the
`config::is_active` function:

``` r
config::is_active("production")
```

## Defaults and Inheritance

The `default` configuration provides a set of values to use when no
named configuration is active. Other configurations automatically
inherit all `default` values so need only define values specialized for
that configuration. For example, in this configuration the `production`
configuration doesn’t specify a value for `trials` so it will be read
from the `default` configuration:

**config.yml**

``` yaml
default:
  trials: 5
  dataset: "data-sampled.csv"
  
production:
  dataset: "data.csv"
```

All configurations automatically inherit from the “default”
configuration. Configurations can also inherit from one or more other
named configurations. For example, in this file the `production`
configuration inherits from the `test` configuration:

**config.yml**

``` yaml
default:
  trials: 5
  dataset: "data-sampled.csv"

test:
  trials: 30
  dataset: "data-test.csv"
  
production:
  inherits: test
  dataset: "data.csv"
```

## Configuration Files

By default configuration data is read from a file named **config.yml**
within the current working directory (or parent directories if no config
file is found in the initially specified directory).

You can use the `file` argument of `config::get` to read from an
alternate location. For example:

``` r
config <- config::get(file = "conf/config.yml")
```

If you don’t want to ever scan parent directories for configuration
files then you can specify `use_parent = FALSE`:

``` r
config <- config::get(file = "conf/config.yml", use_parent = FALSE)
```

## R Code

You can execute R code within configuration files by prefacing values
with `!expr`. This could be useful in the case where you want to base
configuration values on environment variables, R options, or even other
config files. For example:

``` yaml
default:
  cores: 2
  debug: true
  server: "localhost:5555"
   
production:
  cores: !expr getOption("mc.cores")
  debug: !expr Sys.getenv("ENABLE_DEBUG") == "1"
  server: !expr config::get("server", file = "/etc/server-config.yml")
```
