# Grimório — como testar no celular

Três caminhos, do mais rápido ao mais completo. Você pode fazer os três com o mesmo repositório.

| Caminho | Tempo até estar no celular | Gera APK? | iPhone? |
|---|---|---|---|
| **1. GitHub Pages (PWA)** | 3 minutos | Não, mas instala na tela inicial | Sim |
| **2. APK pelo GitHub Actions** | 10 minutos | Sim | Não |
| **3. React Native / Expo** | algumas horas | Sim, e vai para as lojas | Sim |

---

## Caminho 1 — o mais rápido: sem APK nenhum

O app é uma página web. Publicada no GitHub Pages, o Android e o iPhone conseguem instalar na tela inicial: vira ícone, abre em tela cheia, sem barra de navegador, e funciona sem internet.

1. Crie um repositório no GitHub (público, para o Pages ser gratuito).
2. Envie os arquivos deste projeto.
3. Vá em **Settings → Pages → Source: Deploy from a branch → main → /(root)**.
4. Aguarde 2 minutos. O endereço será `https://SEU-USUARIO.github.io/NOME-DO-REPO/www/`.
5. Abra esse endereço no celular:
   - **Android (Chrome)**: menu ⋮ → *Instalar aplicativo*
   - **iPhone (Safari)**: botão compartilhar → *Adicionar à Tela de Início*

Para quem só quer usar o app, este caminho resolve. Não passa por loja, não precisa assinar nada, e a atualização é instantânea: você envia um commit, e no próximo abrir o app já está novo.

---

## Caminho 2 — APK de verdade, compilado pelo GitHub

O GitHub Actions compila um APK na nuvem. Você não precisa instalar Android Studio, nem Java, nem nada no seu computador.

### Passo a passo

**1. Crie o repositório**
No GitHub: **New repository** → nome `grimorio` → deixe **público** (repositório público tem minutos de Actions ilimitados; privado tem 2.000 minutos grátis por mês, e cada build gasta uns 6).

**2. Envie os arquivos**
Use **Add file → Upload files** e arraste a pasta inteira. Esta etapa é bem mais fácil no computador — depois dela, tudo o resto funciona pelo celular.

A estrutura precisa ficar assim:

```
grimorio/
├── .github/workflows/android.yml
├── www/
│   ├── index.html
│   ├── manifest.webmanifest
│   ├── sw.js
│   ├── icone-192.png
│   └── icone-512.png
├── resources/icon.png
├── capacitor.config.json
├── package.json
└── .gitignore
```

**3. Rode o build**
Ao enviar os arquivos para a `main`, o build começa sozinho. Para rodar de novo depois: aba **Actions → Gerar APK → Run workflow**. Esse botão funciona pelo navegador do celular.

**4. Baixe o APK**
Quando terminar (uns 5 a 8 minutos), vá na aba **Releases** do repositório. O arquivo `grimorio-N.apk` está lá, com link direto — abra pelo navegador do próprio celular.

> Prefira o Releases ao artefato da aba Actions: o artefato baixa como `.zip` e exige estar logado, o que é chato no celular.

**5. Instale**
O Android vai perguntar se você confia na origem. Vá em **Configurações → Apps → Acesso especial → Instalar apps desconhecidos** e libere o navegador que você usou. Depois é só tocar no arquivo.

### O que esse APK é e não é

É um **APK de depuração**, assinado com a chave automática do Android. Serve perfeitamente para instalar e testar no seu aparelho e no de quem você quiser. Não serve para publicar na Play Store — para isso é preciso um AAB assinado com a sua própria chave, guardada em Secrets do repositório.

### Ícone do app

Para o APK sair com o ícone certo em vez do padrão do Capacitor, adicione ao workflow, logo depois do `npx cap add android`:

```yaml
      - run: npx @capacitor/assets generate --android
```

Ele lê `resources/icon.png` e gera todos os tamanhos.

---

## Caminho 3 — React Native, quando for para as lojas

Aqui o app deixa de ser uma página dentro de uma casca e vira um aplicativo nativo de verdade. É o caminho descrito no documento de arquitetura.

```bash
npx create-expo-app grimorio --template
cd grimorio
npx eas build -p android --profile preview   # gera um APK instalável
npx eas build -p ios --profile preview       # compila iOS sem precisar de Mac
```

O `--profile preview` gera APK em vez de AAB, que é o que você quer para testar. O EAS tem uma cota gratuita mensal de builds; acima dela é pago.

---

## Sobre o iPhone

Não existe equivalente ao APK no iOS. Instalar um app fora da App Store exige a conta de desenvolvedor da Apple (US$ 99 por ano) e assinatura com certificado próprio. Enquanto isso não fizer sentido:

- **PWA pelo Safari** — o caminho 1 funciona bem, com algumas limitações (notificações e armazenamento são mais restritos que no Android).
- **Expo Go** — se for pelo caminho 3, você instala o app Expo Go da App Store e roda o seu projeto dentro dele, sem build nenhum.

---

## Detalhe importante sobre os dados

Este protótipo tem números fictícios escritos no código e guarda tudo na memória: fechou o app, perdeu as metas criadas. É de propósito — é um protótipo de interface.

Antes de usar de verdade, o próximo passo é a persistência. No caminho da página web, `localStorage` ou IndexedDB. No caminho React Native, MMKV para os dados e `expo-secure-store` para qualquer coisa sensível. Só depois disso entra a conexão com os bancos.
