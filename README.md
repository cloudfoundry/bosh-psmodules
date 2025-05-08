# bosh-psmodules

The recommended approach for creating local BOSH Windows stemcells which can be deployed on [Cloud Foundry BOSH](https://bosh.io), is [`stembuild`](https://github.com/cloudfoundry-incubator/stembuild).

[Documentation on how to use `stembuild` can be found here.](https://bosh.io/docs/windows-stemcell-create/)

---

Powershell scripts to set up a Windows VM in a manner appropriate for a BOSH Stemcell.

Used by [stembuild](https://github.com/cloudfoundry-incubator/stembuild) and [bosh-windows-stemcell-builder](https://github.com/cloudfoundry-incubator/bosh-windows-stemcell-builder)

## Testing

Tests are written using the Pester testing framework and must be run in Powershell on a Windows environment.

The test suite for each module currently assumes that the tests are being run with the module as the current working directory.

This requires iterating through the module directories to run all the tests:

```
cd bosh-psmodules
foreach ($module in (Get-ChildItem "./modules").Name) {
  Push-Location "modules/$module"
    $results=Invoke-Pester -PassThru
    if ($results.FailedCount -gt 0) {
      $result += $results.FailedCount
    }
  Pop-Location
}
echo "Failed Tests: $result"
```

If you just need to test a single module, you could do this:

```
cd "bosh-psmodules\module\BOSH.<module>"
Invoke-Pester
```


### Running tests via Concourse

```
# Set as appropriate for your machine
GREENHOUSE_CI_DIR="$HOME/Workspace/github.com/cloudfoundry/greenhouse-ci"
PSMODULES_DIR="$HOME/Workspace/github.com/cloudfoundry/bosh-psmodules"

fly -t bosh-ecosystem execute \
    --tag=windows-nimbus \
    --config "$GREENHOUSE_CI_DIR/tasks/test-units-bosh-psmodules/task.yml" \
    --inputs-from=windows-2019-stemcell/test-bosh-psmodules \
    --input=bosh-psmodules-repo="$PSMODULES_DIR" \
    --input=ci="$GREENHOUSE_CI_DIR"
```

### Running a subset of tests on MacOS

You can use Powershell and MacOS to run the tests that do not require Windows system calls:

```
cd ~/workspace
brew install powershell
git clone --depth 1 --branch 4.4.0 git@github.com:pester/Pester.git
pwsh
Import-Module ./Pester/Pester.psm1
cd stembuild/module/BOSH.<module>
Invoke-Pester
```

## Debugging

You can debug powershell scripts using VSCode. It has some dependencies:
- dotnet runtime `brew install dotnet`
- powershell binary `brew install powershell`
- vscode extensions: `Powershell`, `C#` and `C# Dev Kit` (the latter may not be required)

You can create a launch.json file like:
```json
{
    "version": "0.2.0",
    "configurations": [

        {
            "name": "PowerShell: Run Pester Tests",
            "type": "PowerShell",
            "request": "launch",
            "script": "Invoke-Pester",
            "createTemporaryIntegratedConsole": true,
            "attachDotnetDebugger": true,
            "cwd": "${file}"
        }
    ]
}
```

And you should be able to run tests for a single file using the debug view. If you're missing extensions 
you'll see odd failures such as the wrong CWD leading to import failures.