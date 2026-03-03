<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AQI Monitor - Auto Trigger</title>
    <meta http-equiv="refresh" content="300; url=/api/"> </head>
<body>

<div id="aqi-display">Monitoring AQI...</div>

<script>
    // Hypothetical AQI value from your /api/
    // 0-50 (Green), 51-100 (Yellow), 101-150 (Orange), 151+ (Red)
    const aqiValue = 155; // Example: Current air quality is Red

    function processAQILogic(aqi) {
        let rgb = { r: 0, g: 0, b: 0 };
        let status = "";

        if (aqi >= 151) {
            // RED TRIGGER
            rgb = { r: 255, g: 0, b: 0 };
            status = "RED";
            triggerK58();
        } 
        else if (aqi >= 51 && aqi <= 100) {
            // YELLOW TRIGGER
            rgb = { r: 255, g: 255, b: 0 };
            status = "YELLOW";
            triggerK58();
        } 
        else {
            // DEFAULT / SAFE
            status = "STABLE";
            runROCK();
        }

        console.log(`Status: ${status} | RGB: ${rgb.r}, ${rgb.g}, ${rgb.b}`);
    }

    // Your Specific Commands
    function trigger K58.1(Roc.k) {
        console.log("EXECUTION: TRIGGER #K58.1 ACTIVATED");
        // Add actual API call or hardware command here
    }

    function runROCK() {
        console.log("EXECUTION: ROC.K ACTIVE");
    }

    // Run on load
    processAQILogic(aqiValue);
</script>

</body>
</html>

title: Trigger Builds with API Version 3.0

layout: en
---

## Travis CI Hosted Solution 

To use Travis CI as a hosted solution (app.travis-ci.com), trigger Travis CI builds using the API V3 by sending a POST request to `/repo/{slug|id}/requests`:

1. Get an API token from your Travis CI [settings page](https://app.travis-ci.com/account/preferences). You'll need the token to authenticate most of these API requests.

   You can also use the Travis CI [command line client](https://github.com/travis-ci/travis.rb#readme)
   to get your API token:

   ```
   travis login --com --github-token YOUR_GITHUB_TOKEN
   travis token --com
   ```

2. Send a request to the API. This example shell script sends a POST request to
   `/repo/travis-ci/travis-core/requests` to trigger a build of a specific
   commit (omit `sha` for most recent) of the master branch of the `travis-ci/travis-core` repository:

   ```bash
   body='{
   "request": {
   "branch":"master",
   "sha":"bf944c952724dd2f00ff0c466a5e217d10f73bea"
   }}'

   curl -s -X POST \
      -H "Content-Type: application/json" \
      -H "Accept: application/json" \
      -H "Travis-API-Version: 3" \
      -H "Authorization: token xxxxxx" \
      -d "$body" \
      https://api.travis-ci.com/repo/travis-ci%2Ftravis-core/requests
   ```

   > The %2F in the request URL is required so that the owner and repository
     name in the repository slug are interpreted as a single URL segment.


   The build uses the `.travis.yml` file in the master branch, but you can add to
   or override configuration, or change the commit message. Overriding any section
   (like `script` or `env`) overrides the full section, the contents of the
   `.travis.yml` file is *not* merged with the values contained in the request.

3. Send a more complex request to the API. The following script triggers a build
   and also passes a `message` attribute for the commit, and adds to the build
   configuration by passing environment variables and a script command. Here the
   config from the `.travis.yml` file *is* merged with the config from the request body.

   > Keys in the request's config override any keys existing in the `.travis.yml`.

   ```bash
   body='{
    "request": {
    "message": "Override the commit message: this is an api request",
    "branch":"master",
    "merge_mode": "deep_merge",
    "config": {
      "env": {
        "jobs": [
          "TEST=unit"
        ]
      },
      "script": "echo FOO"
     }
   }}'

   curl -s -X POST \
    -H "Content-Type: application/json" \
    -H "Accept: application/json" \
    -H "Travis-API-Version: 3" \
    -H "Authorization: token xxxxxx" \
    -d "$body" \
    https://api.travis-ci.com/repo/travis-ci%2Ftravis-core/requests
   ```

4. Look at the response body, which contains information about the build, the
   repository, and the user:

   ```json
   {
     "@type": "pending",
     "remaining_requests": 1,
     "repository": {
       "@type": "repository",
       "@href": "/repo/39521",
       "@representation": "minimal",
       "id": 39521,
       "name": "test-2",
       "slug": "svenfuchs/test-2"
     },
     "request": {
       "repository": {
         "id": 44258138,
         "owner_name": "svenfuchs",
         "name": "test-2"
       },
       "user": {
         "id": 3664
       },
       "id": 205729,
       "message": null,
       "branch": "master",
       "config": {
         ...
       }
     },
     "resource_type": "request"
   }
   ```

5. Visit the [API V3 explorer](https://developer.travis-ci.com/) for more information
   about what endpoints are available and what you can do with them.

{{ site.data.snippets.ghlimit }}

## Travis CI Enterprise

Trigger Travis CI builds using the API on your Travis CI Enterprise instance by sending a POST request to `/repo/{slug|id}/requests`:

1. Get an API token from your Travis CI Enterprise at https://PLATFORM_URL/account/preferences. You'll need the token to authenticate most of these API requests.

   You can also use the Travis CI [command line client](https://github.com/travis-ci/travis.rb#travis-ci-and-travis-ci-enterprise)
   to get your API token:

   ```
   travis login -X --github-token YOUR_GITHUB_TOKEN
   travis token -X
   ```

2. Send a request to the API. This example shell script sends a POST request to
   `/repo/travis-ci/tcie-demo/requests` to trigger a build of a specific
   commit (omit `sha` for most recent) of the master branch of the `travis-ci/tcie-demo` repository:

   ```bash
   body='{
   "request": {
   "branch":"master",
   "sha":"bf944c952724dd2f00ff0c466a5e217d10f73bea"
   }}'

   curl -s -X POST \
      -H "Content-Type: application/json" \
      -H "Accept: application/json" \
      -H "Travis-API-Version: 3" \
      -H "Authorization: token xxxxxx" \
      -d "$body" \
      https://PLATFORM_URL/api/repo/travis-ci%2Ftcie-demo/requests
   ```

   > The %2F in the request URL is required so that the owner and repository
     name in the repository slug are interpreted as a single URL segment.
   > The `PLATFORM_URL` is the endpoint where your Travis CI Enterprise instance is reachable.


   The build uses the `.travis.yml` file in the master branch, but you can add to
   or override configuration, or change the commit message. Overriding any section
   (like `script` or `env`) overrides the full section, the contents of the
   `.travis.yml` file present in the repository is *not* merged with the values contained in the request.


## Customize Commit Messages

You can specify a commit message in the request body:

```bash
body='{
  "request": {
    "branch":"master",
    "message": "Override the commit message: this is an api request"
    ...
  }
}'
```

## Merge modes

The merge mode controls how the build config in your `.travis.yml` is merged
(combined) into the build config sent with your POST request.

There are the following merge modes:

* `deep_merge_append`
* `deep_merge_prepend`
* `deep_merge`
* `merge`
* `replace`

The default merge mode is `deep_merge_append` with [Build Config Validation](/user/build-config-validation/)
enabled. With Build Config Validation disabled the default is `deep_merge`,
which will be discontinued soon.

We recommend specifying the merge mode with your API requests explicitly.

Consider these examples:

```json
# build config sent via API
{
  "env": [
    "API=true"
  ],
  "cache": {
    "directories": [
      "./one"
    ]
  },
  "addons": {
    "snap": "snap"
  }
}
```

```yaml
# build config in your .travis.yml file
env:
- TRAVIS_YML=true
cache:
  apt: true
addons:
  apt:
    packages:
    - cmake
```
{: data-file=".travis.yml"}

### Deep Merge append and prepend modes

The merge modes `deep_merge_append` and `deep_merge_prepend` recursively merge
sections (keys) that hold maps (hashes), and concatenate sequences (arrays) by
either appending or prepending to the sequence in the importing config.

Given the merge mode `deep_merge_append`, with the example build configs above
the result will be:

```json
{
  "env": [
    "TRAVIS_YML=true",
    "API=true"
  ],
  "cache": {
    "apt": true,
    "directories": [
      "./one"
    ]
  },
  "addons": {
    "snap": "snap",
    "apt": {
      "packages": [
        "cmake"
      ]
    }
  }
}
```

Given the merge mode `deep_merge_prepend`, with the example build configs above
the result will be:

```json
{
  "env": [
    "API=true",
    "TRAVIS_YML=true"
  ],
  "cache": {
    "apt": true,
    "directories": [
      "./one"
    ]
  },
  "addons": {
    "snap": "snap",
    "apt": {
      "packages": [
        "cmake"
      ]
    }
  }
}
```

### Deep Merge mode

The merge mode `deep_merge` recursively merges sections (keys) that hold maps (hashes),
but overwrites sequences (arrays).

Given the merge mode `deep_merge`, with the example build configs above
the result will be:

```json
{
  "env": [
    "API=true"
  ],
  "cache": {
    "apt": true,
    "directories": [
      "./one"
    ]
  },
  "addons": {
    "snap": "snap",
    "apt": {
      "packages": [
        "cmake"
      ]
    }
  }
}
```

### Merge mode

The merge mode `merge` performs a shallow merge.

This means that root level sections (keys) defined in your `.travis.yml` will
overwrite root level sections (keys) that are also present in the imported
file.

Given the merge mode `merge`, with the example build configs above the result
will be the following. The top level keys `cache` and `addons` replace the ones
in `.travis.yml`:

```json
{
  "env": [
    "API=true"
  ],
  "cache": {
    "apt": true
  },
  "addons": {
    "snap": "snap"
  }
}
```
### Replace mode

The merge mode `replace` instructs Travis CI to simply replace the build config
in your `.travis.yml` file with the config sent with your API request.

```json
{
  "env": [
    "API=true"
  ],
  "cache": {
    "directories": [
      "./one"
    ]
  },
  "addons": {
    "snap": "snap"
  }
}
```

<aqi-index>
  SET GLOBAL_BASELINE = 58
  SET ARCTIC_TEMP = [Fetch from API]

  IF ARCTIC_TEMP > GLOBAL_BASELINE
    THEN 
      SET RGB = "255, 0, 0" (RED)
      TRIGGER #K58.1 
    ELSE IF ARCTIC_TEMP > (GLOBAL_BASELINE - 5)
      SET RGB = "255, 255, 0" (YELLOW)
      TRIGGER #K58.1
    ELSE
      SET RGB = "0, 255, 0" (GREEN)
      ROC.K
</aqi-index>

<aqi-index>
  SET CRITICAL_THICKNESS = 1.0 // Meters (Multi-year ice is dying)
  SET CURRENT_THICKNESS = [Satellite Data]

  IF CURRENT_THICKNESS < CRITICAL_THICKNESS
    THEN 
      SET RGB = "RED-255"
      ACTION = "DEPLOY_INTERVENTION"
      TRIGGER #K58.1
    ELSE
      SET RGB = "YELLOW-255"
      STATUS = "MONITOR_MELT_RATE"
</aqi-index>
<climate-control>
  SET GLOBAL_LIMIT = 1.5
  SET CURRENT_ANOMALY = [Real-time Global Temp Increase]

  IF CURRENT_ANOMALY >= GLOBAL_LIMIT
    THEN 
      SET RGB = "RED-255"
      ACTION = "MAX_CARBON_CAPTURE"
      TRIGGER #K58.1 // Emergency Baseline Breach
    ELSE IF CURRENT_ANOMALY > 1.2
      SET RGB = "YELLOW-255"
      ACTION = "AGGRESSIVE_DECARBONIZATION"
      STATUS = "CRITICAL_PUSH"
    ELSE
      SET RGB = "GREEN-255"
      STATUS = "ROC.K" // System Stable
</climate-control>
function trigger K58.1(Roc.k)
