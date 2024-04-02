# Use MATLAB with [Codecov](https://codecov.io)

This example shows how to run MATLAB&reg; tests, produce a code coverage report, and upload the report to Codecov. The repository includes these files.

| **File Path**                        | **Description**                                                                                                                                       |
|--------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| `source/quadraticSolver.m` | A function that solves quadratic equations                                                                                            |
| `tests/SolverTest.m`      | A class that tests the `quadraticSolver` function                                                                                          |
| `runMyTests.m`      | A script that creates a test suite and a test runner that outputs a Cobertura code coverage report                                                                                          |
| `azure-pipelines.yml`                | A configuration example for [Azure&reg; DevOps](https://marketplace.visualstudio.com/items?itemName=MathWorks.matlab-azure-devops-extension) |
| `.circleci/config.yml`               | A configuration example for [CircleCI&reg;](https://circleci.com/orbs/registry/orb/mathworks/matlab)
| `.github/workflows/workflow.yml`     | A configuration example for [GitHub&reg; Actions](https://github.com/matlab-actions)

## Produce and Publish Coverage Reports
Each of these pipeline definitions does four things:

1) Install the latest MATLAB release on a Linux&reg;-based build agent.
2) Run all MATLAB tests in the root of your repository, including its subfolders.
3) Produce a code coverage report in Cobertura XML format for the `source` folder.
4) Upload the produced artifact to Codecov.

### Azure DevOps

```yml
pool:
  vmImage: ubuntu-latest
steps:
  - task: InstallMATLAB@1
  - task: RunMATLABTests@1
    inputs:
      sourceFolder: source
      codeCoverageCobertura: coverage.xml
  - script: |
      # download Codecov CLI
      curl -Os https://cli.codecov.io/latest/linux/codecov
      
      # integrity check
      curl https://keybase.io/codecovsecurity/pgp_keys.asc | gpg --no-default-keyring --keyring trustedkeys.gpg --import # One-time step  
      curl -Os https://cli.codecov.io/latest/linux/codecov
      curl -Os https://cli.codecov.io/latest/linux/codecov.SHA256SUM
      curl -Os https://cli.codecov.io/latest/linux/codecov.SHA256SUM.sig
      gpgv codecov.SHA256SUM.sig codecov.SHA256SUM

      # upload to Codecov 
      shasum -a 256 -c codecov.SHA256SUM
      sudo chmod +x codecov
      ./codecov upload-process -t $(CODECOV_TOKEN)
```

### CircleCI

```yml
version: 2.1
orbs:
  matlab: mathworks/matlab@1
  codecov: codecov/codecov@4
jobs:
  build:
    machine:
      image: ubuntu-2204:current
    steps:
      - checkout
      - matlab/install
      - matlab/run-tests:
          source-folder: source
          code-coverage-cobertura: coverage.xml
      - codecov/upload: 
          file: coverage.xml
```

### GitHub Actions

```yml
name: Example
on: push
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: matlab-actions/setup-matlab@v2
      - uses: matlab-actions/run-tests@v2
        with:
          source-folder: source
          code-coverage-cobertura: coverage.xml
      - uses: codecov/codecov-action@v4
        with:
          file: coverage.xml
```

## Links
- [Community Boards](https://community.codecov.io)
- [Support](https://codecov.io/support)
- [Documentation](https://docs.codecov.io)
- [Continuous Integration with MATLAB and Simulink](https://www.mathworks.com/solutions/continuous-integration.html)

## Contact Us
If you have any questions or suggestions, please contact MathWorks&reg; at [continuous-integration@mathworks.com](mailto:continuous-integration@mathworks.com).
