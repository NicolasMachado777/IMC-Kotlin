Integrantes: RM 552493 - Nicolas Machado // RM 552407 - José Roberto Claudino

<img width="404" height="906" alt="image" src="https://github.com/user-attachments/assets/55d96fbb-52bf-4908-96b9-06bafa5d9700" />


<img width="416" height="895" alt="image" src="https://github.com/user-attachments/assets/935cb48d-cc84-4af8-8d8e-b8a7f6b23a4c" />


<img width="410" height="906" alt="image" src="https://github.com/user-attachments/assets/c8e35d3a-1767-4efc-8965-f59ef1757931" />

***

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
