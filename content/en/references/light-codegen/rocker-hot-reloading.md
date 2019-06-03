With support for rocker hot reloading, light-codegen can load customized templates and render classes so that each organization can generate content specific to their own frameworks built on the top of light-4j frameworks.

## Usage

To enable rocker hot reloading, extra classpath setting is needed. A script `codegen-cli/codegen.sh` is provided to help the code generation with hot reloading.

#### MacOS
1. Create rocker compiler config and setting the directory of customized templates (or change the codegen.sh directly), for example `src/test/resources/externalTemplates/`
    ```bash
    cd $workDir
    
    echo "generate config $workDir/rocker-compiler.conf"
    echo "rocker.template.dir=src/test/resources/externalTemplates/" > rocker-compiler.conf
    ```
    The $workDir must contain the `codegen-cli.jar`
    
    This step can be skipped if customized rocker templates locate at the directory of `codegen-cli.jar`.

2. Using `codegen.sh` instead of `java -jar` to execute codegen-cli.jar
    ```bash
    cd $workDir
    
    .././codegen.sh -r -f openapi -o /Users/workspace/test -m /Users/swagger/petstore/1.0.0/openapi.json -c /Users/swagger/petstore/1.0.0/config.json
    ```
    This `-r` means rocker hot reloading is enabled

#### Windows (PowerShell IES)
1. Create rocker compiler config and setting the directory of customized templates (or change the codegen.ps1 directly), for example `src/test/resources/externalTemplates/`
    ```bash
    cd $workDir
    
    Write-Host "generate config $compilerConf"
    $escapedDir = $workDir -replace '\\', '/'
    Set-Content -Path src/test/resources/externalTemplates/ -Value "rocker.template.dir=$escapedDir"
    ```
    The $workDir must contain the `codegen-cli.jar`
    
    This step can be skipped if customized rocker templates locate at the directory of `codegen-cli.jar`.

2. Using `codegen.ps1` instead of `java -jar` to execute codegen-cli.jar
    ```bash
    cd $workDir
    
    .././codegen.ps1 -r -f openapi -o /Users/workspace/test -m /Users/swagger/petstore/1.0.0/openapi.json -c /Users/swagger/petstore/1.0.0/config.json
    ```
    This `-r` means rocker hot reloading is enabled
