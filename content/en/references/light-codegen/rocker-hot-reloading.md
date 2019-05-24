With support for rocker hot reloading, light-codegen can load customized templates and render classes so that each organization can generate content specific to their own frameworks built on the top of light-4j frameworks.

## Usage

To enable rocker hot reloading, extra classpath setting is needed. A script `codegen-cli/codegen.sh` is provided to help the code generation with hot reloading.

1. Create rocker compiler config and setting the directory of customized templates, for example `src/test/resources/externalTemplates/`
    ```bash
    cd light-codegen/codegen-cli/target
    
    echo "generate config ${CURRENT_DIR}/rocker-compiler.conf"
    echo "rocker.template.dir=src/test/resources/externalTemplates/" > rocker-compiler.conf
    ```
    This step can be skipped if customized rocker templates locate at current directory.

2. Using `codegen.sh` instead of `java -jar` to execute codegen-cli.jar
    ```bash
    cd light-codegen/codegen-cli/target
    
    .././codegen.sh -r -f openapi -o /Users/workspace/test -m /Users/swagger/petstore/1.0.0/openapi.json -c /Users/swagger/petstore/1.0.0/config.json
    ```
    This `-r` means rocker hot reloading is enabled
