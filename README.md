# Android IMC App

Calculadora de **IMC (Índice de Massa Corporal)** feita em **Kotlin** com **Jetpack Compose (Material 3)**. O usuário informa **peso (kg)** e **altura (cm)**; o app calcula o IMC e exibe a **classificação** correspondente.

> **Atenção:** Este projeto considera **altura em centímetros**.  
> Ex.: 1,70 m → **170** (e não 1.70).

***

## 🧭 Sumário

*   Tecnologias
*   Arquitetura e Estrutura
*   Tema (Material 3)
*   Lógica de Negócio (IMC)
*   UI e Fluxo da Tela
*   Recursos (cores e imagens)
*   AndroidManifest
*   Como Executar
*   Testes Unitários
*   Melhorias Recomendadas
*   Notas e Cuidados

***

## 🛠 Tecnologias

*   **Kotlin**
*   **Jetpack Compose** (Material 3)
*   **Single-Activity** (`MainActivity`) com UI 100% Compose
*   **Edge-to-Edge** (`enableEdgeToEdge()`)
*   **Theming** com **Light/Dark** e **Dynamic Color** (Android 12+)

***

## 🗂 Arquitetura e Estrutura

Estrutura sugerida (com base nos arquivos enviados):

    app/
     └── src/main/
         ├── AndroidManifest.xml
         ├── java/nicolasmachado777/com/github/android_imc_app/
         │   ├── MainActivity.kt                // Activity + Composable IMCScreen
         │   └── CalculoImc.kt                  // Funções de domínio: calcularImc, determinarClassificacaoIMC
         │
         └── java/nicolasmachado777/com/github/android_imc_app/ui/theme/
             ├── Color.kt                       // Paleta base (Purple*, Pink*)
             ├── Theme.kt                       // AndroidimcappTheme (M3 + Dynamic Color)
             └── Type.kt                        // Typography base (bodyLarge)

### Papéis dos módulos/arquivos

*   **`CalculoImc.kt`**  
    Contém as funções de **domínio** (puras) para cálculo do IMC e determinação da classificação.

*   **`MainActivity.kt`**  
    Inicializa o Compose, aplica o tema `AndroidimcappTheme` e chama o composable **`IMCScreen`**, que contém a UI (header, formulário, botão de calcular e card de resultado).

*   **`ui/theme/`**  
    Implementa o **tema Material 3**, com **dark/light schemes**, **dynamic color** para Android 12+ e **tipografia** base.

***

## 🎨 Tema (Material 3)

Definido em `ui/theme/Theme.kt`, `Color.kt` e `Type.kt`.

*   **Esquemas de cor**:
    *   `LightColorScheme` e `DarkColorScheme` com cores base (`Purple40/80`, `PurpleGrey40/80`, `Pink40/80`);
    *   **Dynamic Color** ativado por padrão em Android 12+:
        ```kotlin
        if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        ```

*   **Tipografia**:
    *   `Typography` define `bodyLarge` e pode ser expandida (títulos, labels etc.).

*   **Uso**:
    ```kotlin
    AndroidimcappTheme {
        Scaffold { innerPadding ->
            IMCScreen(Modifier.padding(innerPadding))
        }
    }
    ```

***

## 🧮 Lógica de Negócio (IMC)

Arquivo: **`CalculoImc.kt`**

### Cálculo do IMC

*   A **altura** é **em centímetros** e convertida para metros dentro da função:

```kotlin
fun calcularImc(altura: Double, peso: Double): Double {
    return peso / (altura / 100).pow(2.0)
}
```

### Classificação

```kotlin
fun determinarClassificacaoIMC(imc: Double): String {
    return if (imc < 18.5) {
        "Abaixo do peso"
    } else if (imc >= 18.5 && imc < 25.0) {
        "Peso Ideal"
    } else if (imc >= 25.0 && imc < 30.0) {
        "Levemente acima do peso"
    } else if (imc >= 30.0 && imc < 35.0) {
        "Obesidade Grau I"
    } else if (imc >= 35.0 && imc < 40.0) {
        "Obesidade Grau II"
    } else {
        "Obesidade Grau III"
    }
}
```

**Faixas:**

| IMC (kg/m²)      | Classificação           |
| ---------------- | ----------------------- |
| `< 18.5`         | Abaixo do peso          |
| `18.5 .. < 25.0` | Peso Ideal              |
| `25.0 .. < 30.0` | Levemente acima do peso |
| `30.0 .. < 35.0` | Obesidade Grau I        |
| `35.0 .. < 40.0` | Obesidade Grau II       |
| `>= 40.0`        | Obesidade Grau III      |

***

## 🧩 UI e Fluxo da Tela

Arquivo: **`MainActivity.kt`** (com o composable `IMCScreen`).

### `MainActivity`

*   Chama `enableEdgeToEdge()` e aplica `AndroidimcappTheme`;
*   Usa `Scaffold` e injeta `IMCScreen`.

### `IMCScreen`

*   **Estado local** com `remember`:
    ```kotlin
    val peso = remember { mutableStateOf("") }
    val altura = remember { mutableStateOf("") }
    val imc = remember { mutableStateOf(0.0) }
    val statusImc = remember { mutableStateOf("") }
    ```

*   **Header** (`Column`):
    *   Altura fixa (180dp), fundo `R.color.vermelho_fiap`;
    *   `Image` (`R.drawable.bmi`) e `Text("Calculadora IMC")`.

*   **Formulário** (`Card`):
    *   Campo **Peso (kg)** — `OutlinedTextField` com teclado **numérico**;
    *   Campo **Altura (cm)** — `OutlinedTextField` com teclado **decimal**;
    *   **Botão CALCULAR**:
        *   Converte textos para `Double` e chama:
            ```kotlin
            imc.value = calcularImc(altura.value.toDouble(), peso.value.toDouble())
            statusImc.value = determinarClassificacaoIMC(imc.value)
            ```

*   **Card de Resultado** (fixado na parte inferior):
    *   Fundo verde (`Color(0xff329F6B)`), título “Resultado”.
    *   (O trecho final do layout veio **cortado**; a expectativa é exibir o **IMC formatado (ex.: 24,22)** e a **classificação** `statusImc.value`.)

> **Sugestão**: formatar o IMC com **duas casas** (`"%.2f".format(imc)`), respeitando a **localização pt-BR** (ver seção de melhorias).

***

## 🧾 Recursos (cores e imagens)

*   `R.color.vermelho_fiap` – cor principal utilizada em textos, bordas e botão.  
    (Definida em `res/values/colors.xml` — **não enviado**, mas referenciado.)
*   `R.drawable.bmi` – imagem/ícone exibido no header.  
    (Colocar em `res/drawable/`.)

> Sem esses recursos, a compilação vai falhar. Certifique-se de adicioná-los.

***

## 📄 AndroidManifest

Arquivo enviado (**`src/main/AndroidManifest.xml`**):

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Androidimcapp">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.Androidimcapp">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

> Observação: Mesmo usando Compose Material 3 com o tema `AndroidimcappTheme` (Kotlin), o Manifest referencia um tema `@style/Theme.Androidimcapp` (XML). Isso é padrão do template. Você pode manter assim, mas o **tema efetivo da UI** vem do **Compose** (em `AndroidimcappTheme { ... }`).

***

## ▶️ Como Executar

1.  **Abrir no Android Studio** (Iguana ou mais recente recomendado).
2.  Garantir dependências do **Compose Material 3** e Kotlin (use o BOM do Compose).
3.  **Sincronizar Gradle**.
4.  Criar/emular um dispositivo (Android 8+ recomendado).
5.  **Run** ▶️.

***

## 🧪 Testes Unitários

Teste o domínio (cálculo e classificação) em **tests JVM**:

```kotlin
import org.junit.Assert.assertEquals
import org.junit.Test
import kotlin.math.abs

class ImcCalculoTest {

    private fun approxEquals(a: Double, b: Double, eps: Double = 1e-2) =
        abs(a - b) < eps

    @Test
    fun `calcularImc com 70kg e 170cm deve ~24_22`() {
        val imc = calcularImc(altura = 170.0, peso = 70.0)
        assert(approxEquals(imc, 24.22))
    }

    @Test
    fun `classificacao Peso Ideal para IMC 22`() {
        val c = determinarClassificacaoIMC(22.0)
        assertEquals("Peso Ideal", c)
    }

    @Test
    fun `classificacao Obesidade Grau III para IMC 41`() {
        val c = determinarClassificacaoIMC(41.0)
        assertEquals("Obesidade Grau III", c)
    }
}
```

***

## 🔧 Melhorias Recomendadas

> Posso aplicar e te devolver o **código completo atualizado** para “copiar e colar”.

1.  **Validação de entrada** (evitar `NumberFormatException`):
    *   Aceitar **vírgula** como separador decimal: `replace(',', '.')`;
    *   Validar valores vazios, negativos e altura 0.
    ```kotlin
    val pesoDouble = peso.value.replace(',', '.').toDoubleOrNull()
    val alturaDouble = altura.value.replace(',', '.').toDoubleOrNull()
    if (pesoDouble == null || alturaDouble == null || alturaDouble <= 0.0) {
        // exibir mensagem amigável ao usuário
        return@Button
    }
    ```
2.  **Formatação local do IMC**:
    ```kotlin
    val imcFormatado = NumberFormat.getNumberInstance(Locale("pt", "BR")).apply {
        maximumFractionDigits = 2
        minimumFractionDigits = 2
    }.format(imc.value)
    ```
3.  **Persistência de estado**:
    *   Trocar `remember` por `rememberSaveable` (sobrevive a rotação/recriação).
4.  **Acessibilidade**:
    *   `contentDescription` em imagens e ícones;
    *   Checar contraste de cores (`vermelho_fiap` vs. fundo).
5.  **Strings em `strings.xml`**:
    *   Facilita i18n e manutenção.
6.  **Refatorar UI em Composables menores** (`IMCHeader`, `IMCForm`, `IMCResultCard`).
7.  **Prevenção de entradas extremas**:
    *   Altura muito baixa (ex.: `< 50 cm`) e muito alta (ex.: `> 250 cm`);
    *   Peso muito baixo/alto — exibir avisos.

***

## 📝 Notas e Cuidados

*   **Altura** deve ser **em centímetros** (ex.: 172).
*   Se o usuário digitar **vírgula** como decimal, sem tratamento, o app pode **crashar** com `NumberFormatException`.
*   Os recursos `R.color.vermelho_fiap` e `R.drawable.bmi` **precisam estar presentes** no projeto.
*   O **tema Compose** (`AndroidimcappTheme`) é quem define as cores/typography da UI, independente do tema XML do Manifest.

***

## 📂 Estrutura da Raiz do Projeto e Função dos Arquivos

Além da pasta `app/` (onde está o código principal), a raiz do projeto Android contém arquivos e diretórios importantes para configuração e build:

| Arquivo/Pasta                 | Função                                                                                                                                     |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **`build.gradle.kts` (raiz)** | Script Gradle **do projeto**. Define repositórios globais, plugins e configurações comuns.                                                 |
| **`settings.gradle.kts`**     | Lista os módulos do projeto (ex.: `include(":app")`) e configurações iniciais do Gradle.                                                   |
| **`gradle.properties`**       | Define propriedades globais (ex.: versão do JVM, flags de build, otimizações).                                                             |
| **`gradlew` / `gradlew.bat`** | Scripts para executar o Gradle **wrapper** (Linux/Mac e Windows). Garantem versão consistente do Gradle sem precisar instalar manualmente. |
| **`gradle/`**                 | Pasta do **Gradle Wrapper** (contém JAR e configs para baixar a versão correta do Gradle).                                                 |
| **`.gitignore`**              | Define arquivos/pastas que não devem ser versionados no Git (ex.: build/, .idea/).                                                         |
| **`README.md`**               | Documentação do projeto (este arquivo).                                                                                                    |
| **`app/build.gradle.kts`**    | Script Gradle **do módulo app**. Define dependências (Compose, Material3, Kotlin), SDKs, e configs específicas do app.                     |
| **`app/proguard-rules.pro`**  | Regras para **ProGuard/R8** (minificação e ofuscação do código em builds release).                                                         |
| **`app/src/test/java/...`**   | Pasta para **testes unitários** (ex.: testes das funções `calcularImc` e `determinarClassificacaoIMC`).                                    |


***

## 1) `ExampleInstrumentedTest.kt` (teste instrumentado)

*   `/** ... */` Comentário de documentação explicando que é um teste que roda em um **dispositivo Android** (emulador ou físico) e linka a documentação oficial.
*   `@RunWith(AndroidJUnit4::class)` Diz ao JUnit para executar esta classe de teste usando o **runner AndroidJUnit4**, próprio para testes instrumentados.
*   `class ExampleInstrumentedTest` Declara a **classe** de testes instrumentados.
*   `@Test` Marca o **método de teste** a seguir.
*   `fun useAppContext()` Nome do teste; a intenção é verificar o **contexto do app**.
*   `val appContext = InstrumentationRegistry.getInstrumentation().targetContext` Obtém o **Context** do app sob teste via **InstrumentationRegistry**.
*   `assertEquals("nicolasmachado777.com.github.android_imc_app", appContext.packageName)` Compara o **packageName** real do app com o esperado. Se for igual, o teste **passa**.

***

## 2) `Theme.kt` (tema Material 3 + Dynamic Color)

*   `package ...ui.theme` Define o **pacote** do arquivo (organização do projeto).

*   `import android.os.Build` Importa a classe **Build** para consultar a versão do Android em runtime.

*   `import androidx.compose.foundation.isSystemInDarkTheme` Função Compose que diz se o sistema está em **tema escuro**.

*   `import androidx.compose.material3.MaterialTheme` Componente que aplica o **tema Material 3** ao conteúdo.

*   `import androidx.compose.material3.darkColorScheme` Cria um **esquema de cores** para modo **escuro**.

*   `import androidx.compose.material3.dynamicDarkColorScheme` Cria esquema **dinâmico** escuro (Material You, Android 12+).

*   `import androidx.compose.material3.dynamicLightColorScheme` Cria esquema **dinâmico** claro (Material You, Android 12+).

*   `import androidx.compose.material3.lightColorScheme` Cria um **esquema de cores** para modo **claro**.

*   `import androidx.compose.runtime.Composable` Marca funções que **compõem UI**.

*   `import androidx.compose.ui.platform.LocalContext` Permite acessar o **Context** Android atual dentro do Compose.

*   `private val DarkColorScheme = darkColorScheme(... )` Define o **esquema de cores do modo escuro** usando `Purple80`, `PurpleGrey80` e `Pink80` como cores primária, secundária e terciária, respectivamente (essas cores vêm do `Color.kt`).

*   `private val LightColorScheme = lightColorScheme(... )` Define o **esquema de cores do modo claro** com `Purple40`, `PurpleGrey40` e `Pink40`. O comentário abaixo mostra outras cores que **poderiam** ser sobrescritas.

*   `@Composable fun AndroidimcappTheme(...)` Função Compose que aplica o **tema do app** ao conteúdo.

*   `darkTheme: Boolean = isSystemInDarkTheme()` Por padrão, o tema **segue** o tema do sistema.

*   `dynamicColor: Boolean = true` **Ativa** cores dinâmicas (Material You) quando possível.

*   `content: @Composable () -> Unit` **Slot** para receber a UI que será “tematizada”.

*   `val colorScheme = when { ... }` Escolhe qual **esquema de cor** usar:
    *   Se `dynamicColor` estiver **ativo** e o Android for **12+** (`Build.VERSION_CODES.S`), cria um esquema **dinâmico** com base no **wallpaper**/sistema: `dynamicDarkColorScheme` se `darkTheme` for verdadeiro; senão `dynamicLightColorScheme`.
    *   Caso contrário, se `darkTheme` for verdadeiro, usa `DarkColorScheme`.
    *   Senão, usa `LightColorScheme`.

*   `MaterialTheme(colorScheme = ..., typography = Typography, content = content)` Aplica o tema com o **esquema de cores escolhido**, a **tipografia** definida em `Type.kt` e **renderiza** o conteúdo.

***

## 3) `Type.kt` (tipografia Material 3)

*   `package ...ui.theme` Define o **pacote**.

*   `import androidx.compose.material3.Typography` Classe de **tipografia** do Material 3.

*   `import androidx.compose.ui.text.TextStyle` Representa um **estilo de texto** (tamanho, altura da linha, etc.).

*   `import androidx.compose.ui.text.font.FontFamily` Define a **família** da fonte.

*   `import androidx.compose.ui.text.font.FontWeight` Define o **peso** da fonte (Normal, Bold, etc.).

*   `import androidx.compose.ui.unit.sp` Unidade **sp** (escala independente) para fontes.

*   `val Typography = Typography( bodyLarge = TextStyle(...) )` Cria a **instância** de tipografia do app:
    *   `fontFamily = FontFamily.Default` Usa a **fonte padrão** do sistema.
    *   `fontWeight = FontWeight.Normal` Peso **normal**.
    *   `fontSize = 16.sp` Tamanho **16sp**.
    *   `lineHeight = 24.sp` **Altura de linha** para legibilidade.
    *   `letterSpacing = 0.5.sp` **Espaçamento** entre caracteres.

*   O comentário abaixo mostra exemplos de como **sobrescrever** outros estilos (ex.: `titleLarge`, `labelSmall`) caso necessário.

***

## 4) `CalculoImc.kt` (regras de negócio do IMC)

*   `package nicolasmachado777.com.github.android_imc_app` Define o **pacote** do arquivo.

*   `import kotlin.math.pow` Importa a função `pow` para fazer **potenciação** em `Double`.

*   `fun calcularImc(altura: Double, peso: Double): Double` Função **pura** que calcula o IMC.
    *   **Importante**: a **altura** é recebida em **centímetros**.
    *   `return peso / (altura / 100).pow(2.0)` Converte a altura de **cm para m** dividindo por 100, eleva ao quadrado e aplica a fórmula **IMC = peso / (altura\_em\_m²)**.

*   `fun determinarClassificacaoIMC(imc: Double): String` Classifica o IMC conforme **faixas**:
    *   `imc < 18.5` → `"Abaixo do peso"`
    *   `18.5 <= imc < 25.0` → `"Peso Ideal"`
    *   `25.0 <= imc < 30.0` → `"Levemente acima do peso"`
    *   `30.0 <= imc < 35.0` → `"Obesidade Grau I"`
    *   `35.0 <= imc < 40.0` → `"Obesidade Grau II"`
    *   `imc >= 40.0` → `"Obesidade Grau III"`

***

## 5) `AndroidManifest.xml` (declarações do app)

*   `<?xml version="1.0" encoding="utf-8"?>` Declara versão e **codificação** do XML.

*   `<manifest xmlns:android="..." xmlns:tools="...">` Define os **namespaces** usados pelos atributos `android:` e `tools:`.

*   `<application ...>` Bloco que define **metadados** do aplicativo:
    *   `android:allowBackup="true"` Permite **backup** automático do app.
    *   `android:dataExtractionRules="@xml/data_extraction_rules"` Regras de **extração de dados** (Android 12+).
    *   `android:fullBackupContent="@xml/backup_rules"` Regras do **backup** (inclui/exclui arquivos).
    *   `android:icon="@mipmap/ic_launcher"` Ícone principal do app.
    *   `android:label="@string/app_name"` Nome do app exibido ao usuário.
    *   `android:roundIcon="@mipmap/ic_launcher_round"` Ícone **redondo** para launchers que exigem esse formato.
    *   `android:supportsRtl="true"` Habilita suporte a **RTL** (idiomas da direita para a esquerda).
    *   `android:theme="@style/Theme.Androidimcapp"` Tema **XML** base aplicado pela Activity (a UI final é tematizada pelo **Compose** em `AndroidimcappTheme`).

*   `<activity android:name=".MainActivity" ...>` Declara a **Activity principal**:
    *   `android:exported="true"` Necessário porque a Activity recebe um **intent-filter MAIN/LAUNCHER** (fica disponível ao sistema).
    *   `android:label="@string/app_name"` Define o **título** da Activity.
    *   `android:theme="@style/Theme.Androidimcapp"` Tema aplicado à Activity (herdado do template).

*   `<intent-filter>` Indica que esta Activity:
    *   `<action android:name="android.intent.action.MAIN" />` É o **ponto de entrada** do app.
    *   `<category android:name="android.intent.category.LAUNCHER" />` Deve ter um **ícone no launcher** (aparece na lista de apps).

***

## 6) `ExampleUnitTest.kt` (teste local/JVM)

*   `package nicolasmachado777.com.github.android_imc_app` Define o **pacote** do teste.

*   `import org.junit.Test` Importa a anotação `@Test`.

*   `import org.junit.Assert.*` Importa funções de **asserção** do JUnit (ex.: `assertEquals`).

*   `/** ... */` Comentário de documentação dizendo que é um **teste local** (roda na **JVM** do computador) e inclui um link da documentação.

*   `class ExampleUnitTest` Declara a **classe** do teste.

*   `@Test` Marca o método como um **caso de teste**.

*   `fun addition_isCorrect()` Nome do teste; aqui ele valida uma soma simples.

*   `assertEquals(4, 2 + 2)` Verifica se `2 + 2` é igual a **4**.

***

## 7) `app/build.gradle.kts` (Gradle do módulo `app`)

**Bloco `plugins`**

*   `alias(libs.plugins.android.application)` Aplica o plugin **Android Application** (AGP), via **Version Catalog**.
*   `alias(libs.plugins.kotlin.android)` Aplica o plugin **Kotlin Android**.
*   `alias(libs.plugins.kotlin.compose)` Ativa o plugin **Compose** para Kotlin (facilita o setup do Compose).

**Bloco `android`**

*   `namespace = "nicolasmachado777.com.github.android_imc_app"` Define o **namespace** do app (gera `R`, `BuildConfig`).
*   `compileSdk = 36` Define a **API** usada para **compilar** (Android 15, no momento).
*   `defaultConfig`:
    *   `applicationId = "nicolasmachado777.com.github.android_imc_app"` ID **único** do app (Play Store).
    *   `minSdk = 24` API **mínima** suportada (Android 7.0).
    *   `targetSdk = 36` API **alvo** para compatibilidade de comportamento.
    *   `versionCode = 1` **Código** interno da versão (inteiro; usado para updates).
    *   `versionName = "1.0"` **Nome** da versão exibido ao usuário.
    *   `testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"` Runner para **testes instrumentados**.
*   `buildTypes`:
    *   `release`:
        *   `isMinifyEnabled = false` **Desliga** minificação/obfuscação (padrão para dev; em produção normalmente ativa).
        *   `proguardFiles(...)` Usa o arquivo **padrão otimizado** e o `proguard-rules.pro` do projeto.
*   `compileOptions`:
    *   `sourceCompatibility = JavaVersion.VERSION_11` Código-fonte **Java 11**.
    *   `targetCompatibility = JavaVersion.VERSION_11` Bytecode **Java 11**.
*   `kotlinOptions`:
    *   `jvmTarget = "11"` **Target** da JVM para o compilador Kotlin.
*   `buildFeatures`:
    *   `compose = true` **Habilita** o Jetpack Compose.

**Bloco `dependencies`**

*   `implementation(libs.androidx.core.ktx)` Extensões Kotlin para **Android Core**.
*   `implementation(libs.androidx.lifecycle.runtime.ktx)` **Lifecycle** com extensões Kotlin.
*   `implementation(libs.androidx.activity.compose)` Integração da **Activity** com Compose.
*   `implementation(platform(libs.androidx.compose.bom))` **BOM** do Compose (sincroniza versões entre artefatos Compose).
*   `implementation(libs.androidx.compose.ui)` Núcleo do **UI Compose**.
*   `implementation(libs.androidx.compose.ui.graphics)` APIs de **gráficos** do Compose.
*   `implementation(libs.androidx.compose.ui.tooling.preview)` Suporte a **Preview** no Android Studio.
*   `implementation(libs.androidx.compose.material3)` Biblioteca **Material 3** para Compose.
*   `testImplementation(libs.junit)` **JUnit** para **testes locais** (JVM).
*   `androidTestImplementation(libs.androidx.junit)` Extensões JUnit para **testes instrumentados**.
*   `androidTestImplementation(libs.androidx.espresso.core)` **Espresso** para testes de **UI instrumentados**.
*   `androidTestImplementation(platform(libs.androidx.compose.bom))` BOM do Compose também no escopo **androidTest**.
*   `androidTestImplementation(libs.androidx.compose.ui.test.junit4)` Testes de **UI com Compose** (JUnit4).
*   `debugImplementation(libs.androidx.compose.ui.tooling)` Ferramentas de **inspeção** do Compose em debug.
*   `debugImplementation(libs.androidx.compose.ui.test.manifest)` **Manifest** auxiliar para testes de UI em debug.

***

## `MainActivity` e `IMCScreen` — explicação linha a linha (texto)

*   `class MainActivity : ComponentActivity()`
    *   Define a Activity principal do app herdando de `ComponentActivity`, base para apps com Jetpack Compose.

*   `override fun onCreate(savedInstanceState: Bundle?)`
    *   Sobrescreve o método chamado quando a Activity é criada.

*   `super.onCreate(savedInstanceState)`
    *   Garante a inicialização padrão da Activity chamando a implementação da superclasse.

*   `enableEdgeToEdge()`
    *   Habilita layout “edge‑to‑edge”, permitindo que o conteúdo ocupe o espaço sob as barras do sistema com tratamento de insets.

*   `setContent {`
    *   Inicia o ambiente de composição do Jetpack Compose para renderizar a UI desta Activity.

*   `AndroidimcappTheme {`
    *   Aplica o tema Material 3 do app (claro/escuro/dinâmico) a todo conteúdo abaixo.

*   `Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->`
    *   Cria uma estrutura base Material com áreas padrão e fornece o `innerPadding` para que o conteúdo respeite barras e elementos do Scaffold.

*   `IMCScreen(modifier = Modifier.padding(innerPadding))`
    *   Desenha a tela do IMC e aplica o padding recebido do Scaffold para evitar sobreposição com barras do sistema.\

***

*   `@Composable`
    *   Indica que a função a seguir compõe UI usando o Jetpack Compose.

*   `fun IMCScreen(modifier: Modifier = Modifier)`
    *   Declara o composable principal da tela do IMC, recebendo um `Modifier` externo opcional.

*   `var peso = remember { mutableStateOf("") }`
    *   Cria estado observável para o texto do peso; inicia vazio e é preservado entre recomposições.

*   `var altura = remember { mutableStateOf("") }`
    *   Cria estado observável para o texto da altura; inicia vazio.

*   `var imc = remember { mutableStateOf(0.0) }`
    *   Cria estado para o valor numérico do IMC calculado; inicia em 0.0.

*   `var statusImc = remember { mutableStateOf("") }`
    *   Cria estado para o texto da classificação do IMC; inicia vazio.

*   `Box( modifier = modifier.fillMaxSize() )`
    *   Contêiner que empilha filhos e expande para preencher toda a tela.

*   `Column( modifier = Modifier.fillMaxWidth() )`
    *   Coluna principal da tela que ocupa toda a largura disponível.

*   `Column( horizontalAlignment = Alignment.CenterHorizontally, modifier = Modifier.fillMaxWidth().height(180.dp).background(colorResource(id = R.color.vermelho_fiap)) )`
    *   Cabeçalho: centraliza conteúdo, ocupa toda a largura, tem 180dp de altura e fundo usando a cor `vermelho_fiap`.

*   `Image( painter = painterResource(id = R.drawable.bmi), contentDescription = "logo", modifier = Modifier.size(100.dp).padding(top = 16.dp) )`
    *   Exibe a imagem `bmi` como logo, com tamanho 100dp e espaçamento superior.

*   `Text( text = "Calculadora IMC", fontSize = 24.sp, color = Color.White, fontWeight = FontWeight.Bold, modifier = Modifier.padding(top = 12.dp, bottom = 24.dp) )`
    *   Mostra o título do app no cabeçalho, com fonte grande, branca e em negrito, com espaçamento acima e abaixo.

*   `Column( modifier = Modifier.fillMaxWidth().padding(horizontal = 32.dp) )`
    *   Início da seção do formulário, ocupando a largura total e com 32dp de padding lateral.

*   `Card( modifier = Modifier.offset(y = (-30).dp).fillMaxWidth(), colors = CardDefaults.cardColors(containerColor = Color(0xfff9f6f6)), elevation = CardDefaults.cardElevation(4.dp) )`
    *   Cartão do formulário deslocado 30dp para cima (sobrepondo parte do header), com fundo claro e leve elevação.

*   `Column(modifier = Modifier.padding(24.dp))`
    *   Área interna do card com 24dp de padding para respiro do conteúdo.

*   `Text( text = "Seus dados", modifier = Modifier.fillMaxWidth(), fontSize = 24.sp, fontWeight = FontWeight.Bold, color = colorResource(id = R.color.vermelho_fiap), textAlign = TextAlign.Center )`
    *   Título da seção do formulário “Seus dados”, centralizado, grande, negrito e com a cor `vermelho_fiap`.

*   `Spacer(modifier = Modifier.height(32.dp))`
    *   Espaço vertical de 32dp separando o título dos próximos campos.

*   `Text( text = "Seu peso", modifier = Modifier.padding(bottom = 8.dp), fontSize = 12.sp, fontWeight = FontWeight.Normal, color = colorResource(id = R.color.vermelho_fiap) )`
    *   Rótulo do campo de **peso**, pequeno e com cor temática.

*   `OutlinedTextField( value = peso.value, onValueChange = { peso.value = it }, modifier = Modifier.fillMaxWidth(), placeholder = { Text(text = "Seu peso em Kg.") }, colors = OutlinedTextFieldDefaults.colors( unfocusedBorderColor = colorResource(id = R.color.vermelho_fiap), focusedBorderColor = colorResource(id = R.color.vermelho_fiap) ), shape = RoundedCornerShape(16.dp), keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number) )`
    *   Campo de texto contornado para **peso**, ocupando toda a largura, com placeholder explicativo, bordas na cor `vermelho_fiap`, cantos arredondados e teclado numérico.

*   `Spacer(modifier = Modifier.height(16.dp))`
    *   Espaço vertical entre o campo de peso e o rótulo da altura.

*   `Text( text = "Sua altura", modifier = Modifier.padding(bottom = 8.dp), fontSize = 12.sp, fontWeight = FontWeight.Normal, color = colorResource(id = R.color.vermelho_fiap) )`
    *   Rótulo do campo de **altura**, pequeno e com cor temática.

*   `OutlinedTextField( value = altura.value, onValueChange = { altura.value = it }, modifier = Modifier.fillMaxWidth(), placeholder = { Text( text = "Sua altura em cm." ) }, colors = OutlinedTextFieldDefaults.colors( unfocusedBorderColor = colorResource(id = R.color.vermelho_fiap), focusedBorderColor = colorResource(id = R.color.vermelho_fiap) ), shape = RoundedCornerShape(16.dp), keyboardOptions = KeyboardOptions( keyboardType = KeyboardType.Decimal ) )`
    *   Campo de texto contornado para **altura**, com placeholder indicando centímetros, bordas temáticas, cantos arredondados e teclado decimal.

*   `Spacer(modifier = Modifier.height(16.dp))`
    *   Espaço vertical entre o campo de altura e o botão de ação.

*   `Button( onClick = { ... }, modifier = Modifier.fillMaxWidth().height(48.dp), shape = RoundedCornerShape(16.dp), colors = ButtonDefaults.buttonColors( containerColor = colorResource(id = R.color.vermelho_fiap) ) )`
    *   Botão principal de ação “CALCULAR”, ocupando toda a largura, com altura de 48dp, cantos arredondados e cor de fundo `vermelho_fiap`.

*   `imc.value = calcularImc( altura = altura.value.toDouble(), peso = peso.value.toDouble() )`
    *   No clique do botão, converte os textos de altura e peso para `Double` e calcula o IMC usando a função de domínio.

*   `statusImc.value = determinarClassificacaoIMC(imc.value)`
    *   Ainda no clique, determina a **classificação** do IMC calculado e atualiza o estado.

*   `Text( text = "CALCULAR", fontWeight = FontWeight.Bold, color = Color.White, fontSize = 14.sp )`
    *   Rótulo do botão em negrito, branco e com tamanho 14sp.

*   `Card( modifier = Modifier.fillMaxWidth().height(200.dp).padding(horizontal = 32.dp, vertical = 24.dp).align(Alignment.BottomCenter), colors = CardDefaults.cardColors(containerColor = Color(0xff329F6B)), elevation = CardDefaults.cardElevation(4.dp) )`
    *   Cartão de **resultado** fixado na parte inferior, com altura 200dp, padding interno e fundo verde `#329F6B`, além de elevação.

*   `Row( verticalAlignment = Alignment.CenterVertically, modifier = Modifier.padding(horizontal = 32.dp).fillMaxSize() )`
    *   Linha interna do card de resultado, centralizando verticalmente os filhos, com padding lateral e ocupando o card todo.

*   `Column( modifier = Modifier.weight(1f) )`
    *   Coluna à esquerda com peso 1 para ocupar o espaço disponível, permitindo que o valor do IMC fique alinhado à direita.

*   `Text( text = "Resultado", color = Color.White, fontSize = 14.sp )`
    *   Cabeçalho “Resultado” no card, com cor branca e tamanho pequeno.

*   `Text( text = statusImc.value, fontWeight = FontWeight.Bold, color = Color.White, fontSize = 20.sp )`
    *   Exibe a **classificação** do IMC em destaque (negrito, branco, 20sp).

*   `Text( text = String.format("%.1f", imc.value), fontWeight = FontWeight.Bold, color = Color.White, fontSize = 36.sp, textAlign = TextAlign.End, maxLines = 1, softWrap = false, style = TextStyle( fontFeatureSettings = "", fontFamily = FontFamily.Default ) )`
    *   Exibe o **valor numérico** do IMC com **uma casa decimal**, em negrito, branco, grande (36sp), alinhado à direita, sem quebra de linha, e com estilo padrão de fonte.
