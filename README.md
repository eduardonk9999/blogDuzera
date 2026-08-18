# duzera blog

Blog pessoal com tema de terminal Linux, construído com Astro 7.

🌐 **Live**: [duzeradev.com.br](https://duzeradev.com.br)

## Stack

- **Framework**: Astro 7
- **Linguagem**: TypeScript
- **Estilo**: CSS Variables
- **Fonte**: JetBrains Mono
- **Deploy**: Vercel
- **Hospedagem**: GitHub

## Features

- ✅ Sistema de posts com Astro Content Collections
- ✅ Rotas dinâmicas
- ✅ Tema visual de terminal Linux
- ✅ Animações CSS (typewriter, blink, fadeIn)
- ✅ Responsivo
- ✅ SEO otimizado

## Comandos

```bash
npm install          # Instala dependências
npm run dev          # Inicia servidor local em localhost:4321
npm run build        # Build de produção para ./dist/
npm run preview      # Preview do build local
```

## Estrutura do Projeto

```
/
├── src/
│   ├── components/      # Componentes reutilizáveis
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   ├── PostCard.astro
│   │   └── TerminalWindow.astro
│   ├── content/         # Posts em Markdown
│   │   └── posts/
│   ├── layouts/         # Layouts do site
│   │   ├── Layout.astro
│   │   └── PostLayout.astro
│   ├── pages/           # Rotas do site
│   │   ├── index.astro
│   │   ├── about.astro
│   │   └── posts/[slug].astro
│   └── styles/          # Estilos globais
│       └── global.css
└── public/              # Assets estáticos
```

## Como adicionar um post

Crie um arquivo `.md` em `src/content/posts/`:

```yaml
---
title: "Título do post"
date: 2026-08-18
description: "Descrição curta"
tags: ["tag1", "tag2"]  # opcional
draft: false            # opcional
---

Conteúdo do post aqui...
```

## Autor

**Eduardo Alves** - [@eduardonk9999](https://github.com/eduardonk9999)

## Licença

MIT
