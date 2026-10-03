# FleetCheck – Build Systems Lab

This project is intentionally incomplete. Follow the worksheet in the order given.

Expected final application output:

```
FleetCheck 1.0
Vehicles loaded: 4
Vehicles requiring service: 2
Average mileage: 37000 km
```

Do not copy the solution POM. The objective is to observe how each build change alters the result.

[ERROR] /Users/danielrodrigues/Documents/MaterialUniversidade/3ºAno/QS/Projetos/Worksheet 4 - Maven amp Gradle-20260928/FleetCheck_Starter/src/main/java/pt/upt/fleetcheck/App.java:[3,39] package com.fasterxml.jackson.core.type does not exist
---

## Evidências – Parte Maven

**Evidence 1** – Erro da compilação inicial (acima). Provocado pelo `import com.fasterxml.jackson.core.type.TypeReference;` (e `com.fasterxml.jackson.databind.ObjectMapper`) em `App.java`, porque a dependência `jackson-databind` não estava declarada no `pom.xml`.

**Pergunta 2 – Porque é que esta falha é melhor do que a do passo 1?**
A falha do passo 1 era de configuração: faltava uma dependência e o código nem compilava. Depois de declarar o Jackson, o build avança até à fase de testes e passa a detetar um defeito de comportamento real (erro de fronteira em `FleetService.needsService`: `>` em vez de `>=`). O build já está a verificar a qualidade do software e não apenas a montá-lo. Defeito corrigido: um veículo que atinge exatamente o intervalo de revisão (10000 km) passa a contar como a precisar de revisão.

**Passo 3 – `mvn dependency:tree`**
`jackson-databind` aparece como dependência direta; `jackson-core` e `jackson-annotations` aparecem por baixo dela como dependências transitivas. O JUnit aparece com scope `test`.

**Evidence 4 – O que mudou com o Shade plugin?**
O JAR por omissão (`fleetcheck-1.0.0.jar`) só contém as classes do FleetCheck e não tem `Main-Class` no manifest, por isso `java -jar` falha. O Shade plugin gera um segundo artefacto (`fleetcheck-1.0.0-all.jar`) que inclui também as classes do Jackson (databind, core, annotations) e escreve `Main-Class: pt.upt.fleetcheck.App` no manifest através do `ManifestResourceTransformer`. O resultado é um "fat JAR" autónomo que corre só com `java -jar`.

**Pergunta 5 – Que pressuposto do ambiente o wrapper removeu?**
O pressuposto de que cada máquina (de cada colega ou do servidor de CI) tem o Maven instalado, no PATH e numa versão compatível. Com o wrapper, a versão exata do Maven fica registada em `.mvn/wrapper/maven-wrapper.properties` e é descarregada automaticamente; basta ter um JDK. O `project.build.outputTimestamp` fixa as datas dentro do JAR, tornando os builds reprodutíveis.

**Evidence 6 – GitHub Actions**
Run com sucesso: https://github.com/DanielRodrigues871/Worksheet4_48102/actions/runs/37137469460 (artefacto `fleetcheck-build` anexado).

**Evidence 7 – Porque é que o SBOM tem componentes que não escrevi?**
O SBOM regista o grafo completo de dependências resolvidas, não só as diretas. Declarei apenas `jackson-databind`, mas o Maven resolveu também as suas dependências transitivas (`jackson-core` e `jackson-annotations`), que fazem parte do software entregue. Um SBOM tem de listar tudo o que vai no produto para permitir rastrear vulnerabilidades e licenças.
