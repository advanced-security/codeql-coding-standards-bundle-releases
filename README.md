# CodeQL Coding Standards Bundle

The CodeQL Coding Standards Bundle is a CodeQL bundle that includes the queries from the matching [CodeQL Coding Standards](https://github.com/github/codeql-coding-standards) project that is be open sourced in July 2022. More information on the CodeQL Coding Standards project can be found in [this](https://github.blog/2022-06-20-adding-support-for-coding-standards-autosar-c-and-cert-c/) blog post.

The queries implement the guidelines specified in the following standards targeting the projects using C++ revision [14](https://www.iso.org/standard/64029.html):
- [AUTOSAR - Guidelines for the use of C++14 language in critical and safety-related systems Release 18-10](https://www.autosar.org/fileadmin/user_upload/standards/adaptive/18-10/AUTOSAR_RS_CPP14Guidelines.pdf)
- [MISRA C++:2008](https://www.misra.org.uk)
- [SEI CERT C++ Coding Standard: Rules for Developing Safe, Reliable, and Secure Systems (2016 Edition)](https://resources.sei.cmu.edu/library/asset-view.cfm?assetID=494932)

The [list of supported rules](./supported_rules_list.csv) lists per standard and rule what query, or queries, implement that rule.

## How to use the bundle

The bundle can be use with the [Github CodeQL Action](https://github.com/github/codeql-action) by preceding the `github/codeql-action/init@v2` step with the following step:

```yaml
- name: Download CodeQL Coding Standards Bundle
  run: |
    gh release download -R advanced-security/codeql-coding-standards-bundle-releases v1.10.0 --pattern 'codeql-coding-standards.tar.gz'
```

The step initializing the Github CodeQL Action using `github/codeql-action/init@v2` can be instructed to use the bundle through the `tools` key and the queries can be specified through the `queries` key as follows:

```yaml
- name: CodeQL Initialize
  uses: github/codeql-action/init@v2
  with:
    tools: codeql-coding-standards.tar.gz
    queries: autosar-default,cert-default
```

The CodeQL Coding Standards Bundle supports the following CodeQL query suites:

- `autosar-default`: All the supported AUTOSAR queries that are not audit queries.
- `autosar-required`: The AUTOSAR queries with obligation *required*, and that are not audit queries.
- `autosar-advisory`: The AUTOSAR queries with obligation *advisory*, and that are not audit queries.
- `autosar-audit`: The AUTOSAR queries that are audit queries. An audit query provides information that can aid in a manual review of a guideline with enforcement *non-automated*.
- `cert-default`: All the supported CERT queries.

## Reporting issues

This project is providing a deployment option for the coding standards queries, but is not in any way involved with the implementation details of those queries.
Feel free to open issues encountered when deploying this bundle.

However, any issues encountered (e.g., false positives, false negatives, performance) when applying the coding standards queries to your projects should be reported in the CodeQL Coding Standards [issue tracker](https://github.com/github/codeql-coding-standards/issues) when that has been made available.

## Troubleshooting

An elaborate user manual will be provided when the CodeQL Coding Standards is open sourced.
However the following errors might be troubleshooted if encountered.

### Error: Code Scanning could not process the submitted SARIF file

The error can occur using the action `github/codeql-action/analyze@v2` or `github/codeql-action/upload-sarif@v2` that uploads the results of the CodeQL analysis with the following reason

`rejecting SARIF, as there are more results per run than allowed (25271 > 25000)`

This can occur when the CodeQL Coding Standard queries are used on a project that doesn't adhere to the standard resulting a one or more queries returning a large number of alerts.
The following steps can be used to troubleshoot the issue:

1. When using the `github/codeql-action/analyze@v2`, disable the automatic uploading of the SARIF file as follows:
   ```yaml
   - name: CodeQL Analyze
     uses: github/codeql-action/analyze@v2
     with:
        upload: "false"
   ```
2. Upload the SARIF file with the `actions/` as follows:
   ```yaml
   - name: Upload SARIF
     uses: actions/upload-artifact@v2
     with:
        name: results
        path: "../results"
   ```
3. Analyze the SARIF file in [Visual Studio Code](https://code.visualstudio.com/) using the [SARIF Viewer](https://marketplace.visualstudio.com/items?itemName=MS-SarifVSCode.sarif-viewer) extension. The *rules* tab of the SARIF Viewer gives a breakdown per rule and the number of alerts.
4. Note down the rule id (of the form `cpp/autosar/...`) of the rules with a high number of alerts.
5. Revert the above changes to return to the regular workflow.
6. Create a CodeQL query suite that excludes the identified rule(s). More information on creating CodeQL query suites can be found at [Creating CodeQL query suites](https://codeql.github.com/docs/codeql-cli/creating-codeql-query-suites/). The following is an example for AUTOSAR that excludes the rule `cpp/autosar/undocumented-user-defined-type`:
   ```yaml
   - description: AUTOSAR C++14 Guidelines 19-11 (Customized)
   - import: codeql-suites/autosar-default.qls
     from: autosar-cpp-coding-standards
   - exclude:
     id:
        - cpp/autosar/undocumented-user-defined-type
   ```
7. Add the CodeQL query suite to the repository and refer to it in the `github/codeql-action/init@v2` step. The following example assumes the CodeQL query suite is stored at `.github/code-scanning/autosar.qls`:
   ```yaml
   - name: CodeQL Initialize
     uses: github/codeql-action/init@v2
     with:
        tools: codeql-coding-standards.tar.gz
        queries: cert-default,.github/code-scanning/autosar.qls