# Generate SARIF from Fortify on Demand

Build secure software fast with [Fortify](https://www.microfocus.com/en-us/solutions/application-security). Fortify offers end-to-end application security solutions with the flexibility of testing on-premises and on-demand to scale and cover the entire software development lifecycle.  With Fortify, find security issues early and fix at the speed of DevOps. 

This GitHub Action invokes the Fortify Software Security Center (SSC) API to generate a SARIF log file of Static Application Security Testing (SAST) results. The SARIF output is optimized for subsequent import into GitHub to display vulnerabilities in the Security Code Scanning Alerts.

## Usage

The primary use case for this action is after completion of a Fortify SCA or ScanCentral SAST scan. See the [Fortify ScanCentral Scan](https://github.com/marketplace/actions/fortify-scancentral-scan) action for more details on how to initiate a Fortify ScanCentral SAST scan in your workflow. The following sample workflow demonstrates steps to import results from SSC into GitHub Security Code Scanning:

```yaml
name: Import SSC SAST Results
on: [workflow dispatch]
      
jobs:                                                  
  Import-SSC-SAST:
    runs-on: ubuntu-latest

    steps:
      # Pull SAST issues from Fortify Software Security Center and generate SARIF output
      - name: Download Results
        uses: fortify/gha-ssc-generate-sarif@master
        with:
          ssc_base_url: ${{ secrets.SSC_BASE_URL }}
          ssc_auth_token: ${{ secrets.SSC_AUTHTOKEN_DECODED }}
          ssc_version_id: ${{ secrets.SSC_VERSION_ID }}
      
      # Import Fortify Software Security Center results to GitHub Security Code Scanning
      - name: Import Results
        uses: github/codeql-action/upload-sarif@v1
        with:
          sarif_file: ./gh-fortify-sast.sarif

```

For sample workflows implementing this and other Fortify actions, see:
  * [EightBall](https://github.com/fortify/gha-sample-workflows-eightball/tree/master/.github/workflows)

### Considerations

* Issues that are suppressed or hidden in SSC are ignored.
* SARIF is designed specifically for SAST findings, so this action ignores FoD Dynamic (DAST), Mobile (MAST) and Open Source/Software Composition (OSS/SCA) issues.
* GitHub Code Scanning currently supports SARIF files with up to 1,000 issues. If the SSC application version contains more than 1,000 issues, this action will abort. Consider creating a dedicated filter set on SSC that produces less
than 1,000 issues, in combination with the `ssc_vuln_filter_set_id` option.


## Inputs

### `ssc_base_url`
**Required** The base URL for the Fortify Software Security Center instance where your data resides.

### `ssc_auth_token` OR `ssc_user_name` + `ssc_password`
**Required** Credentials for authenticating to Software Security Center. Strongly recommend use of GitHub Secrets for credential management.  Use a `CI Token` if you wish to use token-based authentication.

### `ssc_version_id` OR `ssc_version_name`
**Required** The target SSC application version ID or name to pull SAST issues from. When specifying an application version name, it must be in the format `<application name>:<version name>`.

### `ssc_vuln_filter_set_id`
**Optional** ID of the SSC filter set from which to pull SAST issues.

### `sarif_output`
**Optional** The location of generated SARIF output, defaults to `${GITHUB_WORKSPACE}/gh-fortify-sast.sarif`

## Outputs
SARIF log file that is optimized for subsequent import and viewing in GitHub Security Code Scanning
