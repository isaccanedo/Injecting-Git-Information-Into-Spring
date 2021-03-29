# Injetando informações Git no Spring

# 1. Visão Geral
Neste tutorial, vamos mostrar como injetar informações de repositório Git em um aplicativo baseado em Spring Boot construído com Maven.

Para fazer isso, usaremos maven-git-commit-id-plugin - uma ferramenta útil criada exclusivamente para este propósito.

# 2. Dependências Maven
Vamos adicionar um plugin a uma seção <plugins> do nosso arquivo pom.xml do nosso projeto:

```
<plugin>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
    <version>2.2.1</version>
</plugin
```

Lembre-se de que este plugin requer pelo menos a versão 3.1.1 do Maven.

# 3. Configuração
O plugin tem muitos sinalizadores e atributos convenientes que expandem sua funcionalidade. Nesta seção, descreveremos brevemente alguns deles. Se você quiser conhecer todos eles, visite a página de maven-git-commit-id-plugin, e se você quiser ir direto para o exemplo, vá para a seção 4.

Os trechos a seguir contêm exemplos de atributos de plug-in; especifique-os em uma seção <configuration> </configuration> de acordo com suas necessidades.

### 3.1. Repositório ausente
Você pode configurá-lo para omitir erros se o repositório Git não for encontrado:

```
<failOnNoGitDirectory>false</failOnNoGitDirectory>
```

### 3.2. Localização do repositório Git
Se você deseja especificar o local do repositório .git personalizado, use o atributo dotGitDirectory:

```
<dotGitDirectory>${project.basedir}/submodule_directory/.git</dotGitDirectory>
```

### 3.3. Arquivo de saída
Para gerar o arquivo de propriedades com um nome e / ou diretório personalizado, use a seguinte seção:

```
<generateGitPropertiesFilename>
    ${project.build.outputDirectory}/filename.properties
</generateGitPropertiesFilename>
```

### 3.4. Verbosidade

Para um registro mais generoso, use:

```
<verbose>true</verbose>
```

### 3.5. Geração de arquivo de propriedades
Você pode desativar a criação de um arquivo git.properties:

```
<generateGitPropertiesFile>false</generateGitPropertiesFile>
```

### 3.6. Prefixo das propriedades
Se você deseja especificar um prefixo de propriedade personalizada, use:

```
<prefix>git</prefix>
```

### 3.7. Apenas para repositório pai
Ao trabalhar com projeto com submódulos, definir esta sinalização garante que o plug-in funcione apenas para o repositório pai:

```
<runOnlyOnce>true</runOnlyOnce>
```

### 3.8. Exclusão de propriedades
Você pode querer excluir alguns dados confidenciais, como informações do usuário do repositório:

```
<excludeProperties>
    <excludeProperty>git.user.*</excludeProperty>
</excludeProperties>
```

### 3.9. Inclusão de Propriedades
Incluir apenas dados especificados também é possível:

```
<includeOnlyProperties>    
    <includeOnlyProperty>git.commit.id</includeOnlyProperty>
</includeOnlyProperties>
```

# 4. Aplicativo de amostra
Vamos criar um controlador REST de amostra, que retornará informações básicas sobre nosso projeto.

Criaremos o aplicativo de amostra usando Spring Boot. Se você não sabe como configurar um aplicativo Spring Boot, consulte o artigo introdutório: Configure a Spring Boot Web Application.

Nosso aplicativo consistirá em 2 classes: Application e CommitIdController

### 4.1. Inscrição

CommitIdApplication servirá como uma raiz de nosso aplicativo:

```
@SpringBootApplication(scanBasePackages = { "com.isaccanedo.git" })
public class CommitIdApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(CommitIdApplication.class, args);
    }
 
    @Bean
    public static PropertySourcesPlaceholderConfigurer placeholderConfigurer() {
        PropertySourcesPlaceholderConfigurer propsConfig 
          = new PropertySourcesPlaceholderConfigurer();
        propsConfig.setLocation(new ClassPathResource("git.properties"));
        propsConfig.setIgnoreResourceNotFound(true);
        propsConfig.setIgnoreUnresolvablePlaceholders(true);
        return propsConfig;
    }
}
```

Além de configurar a raiz de nosso aplicativo, criamos o bean PropertyPlaceHolderConfigurer para que possamos acessar o arquivo de propriedades gerado pelo plugin.

Também definimos alguns sinalizadores, para que o aplicativo funcione sem problemas mesmo se o Spring não puder resolver o arquivo git.properties.

### 4.2. Controlador

```
@RestController
public class CommitInfoController {

    @Value("${git.commit.message.short}")
    private String commitMessage;

    @Value("${git.branch}")
    private String branch;

    @Value("${git.commit.id}")
    private String commitId;

    @RequestMapping("/commitId")
    public Map<String, String> getCommitId() {
        Map<String, String> result = new HashMap<>();
        result.put("Commit message",commitMessage);
        result.put("Commit branch", branch);
        result.put("Commit id", commitId);
        return result;
    }
}
```

Como você pode ver, estamos injetando propriedades do Git nos campos de classe.

Para ver todas as propriedades disponíveis, consulte o arquivo git.properties ou a página do autor no Github. Também criamos um endpoint simples que, na solicitação HTTP GET, responderá com um JSON contendo valores injetados.

### 4.3. Maven Entry
Primeiro, vamos configurar as etapas de execução a serem realizadas pelo plug-in, além de qualquer outra propriedade de configuração que consideramos útil:

```
<plugin>
    <groupId>pl.project13.maven</groupId>
    <artifactId>git-commit-id-plugin</artifactId>
    <version>2.2.1</version>
    <executions>
        <execution>
            <id>get-the-git-infos</id>
            <goals>
                <goal>revision</goal>
            </goals>
        </execution>
        <execution>
            <id>validate-the-git-infos</id>
            <goals>
                <goal>validateRevision</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- ... -->
    </configuration>
</plugin>
```

Para que nosso código funcione corretamente, precisamos terminar com um arquivo git.properties em nosso classpath. Para isso, temos duas opções.

A primeira é deixar que o plugin gere o arquivo. Podemos especificar isso definindo a propriedade de configuração generateGitPropertiesFile com um valor verdadeiro:

```
<configuration>
    <generateGitPropertiesFile>true</generateGitPropertiesFile>
</configuration>
```

A segunda opção é incluir um arquivo git.properties na pasta de recursos. Podemos incluir apenas as entradas que usaremos em nosso projeto:

```
# git.properties
git.tags=${git.tags}
git.branch=${git.branch}
git.dirty=${git.dirty}
git.remote.origin.url=${git.remote.origin.url}
git.commit.id=${git.commit.id}
git.commit.id.abbrev=${git.commit.id.abbrev}
git.commit.id.describe=${git.commit.id.describe}
git.commit.id.describe-short=${git.commit.id.describe-short}
git.commit.user.name=${git.commit.user.name}
git.commit.user.email=${git.commit.user.email}
git.commit.message.full=${git.commit.message.full}
git.commit.message.short=${git.commit.message.short}
git.commit.time=${git.commit.time}
git.closest.tag.name=${git.closest.tag.name}
git.closest.tag.commit.count=${git.closest.tag.commit.count}
git.build.user.name=${git.build.user.name}
git.build.user.email=${git.build.user.email}
git.build.time=${git.build.time}
git.build.host=${git.build.host}
git.build.version=${git.build.version}
```

O Maven substituirá os marcadores de posição pelos valores apropriados.

Nota: Alguns IDEs não funcionam bem com este plug-in e podem lançar um erro de "referência de espaço reservado circular" no bootstrap quando definimos as propriedades como fizemos acima.

Após inicializar e solicitar localhost:8080/commitId, você pode ver um arquivo JSON com uma estrutura semelhante à seguinte:

```
{
    "Commit id":"7adb64f1800f8a84c35fef9e5d15c10ab8ecffa6",
    "Commit branch":"commit_id_plugin",
    "Commit message":"Merge branch 'master' into commit_id_plugin"
}
```

# 5. Integração com Spring Boot Actuator
Você pode usar o plugin com Spring Actuator facilmente.

Como você pode ler na documentação, GitInfoContributor selecionará o arquivo git.properties, se disponível. Portanto, com a configuração do plug-in padrão, as informações do Git serão retornadas ao chamar /info endpoint:

```
{
  "git": {
    "branch": "commit_id_plugin",
    "commit": {
      "id": "7adb64f",
      "time": "2016-08-17T19:30:34+0200"
    }
  }
}
```

# 6. Conclusão
Neste tutorial, mostramos o básico do uso de maven-git-commit-id-plugin e criamos um aplicativo Spring Boot simples, que faz uso de propriedades geradas pelo plugin.

A configuração apresentada não cobre todos os sinalizadores e atributos disponíveis, mas cobre todos os fundamentos necessários para começar a trabalhar com este plugin.