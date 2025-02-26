# Fetching GitHub Content in Astro + React Project

This guide explains how to fetch contents from a GitHub repository into a Vercel-deployed Astro + React project that's in a separate GitHub repo.

## Setup

1. Create an API route in your Astro project:
   - Add a file `src/pages/api/github-content.ts`

2. Install necessary packages:
   ```
   npm install @octokit/rest
   ```

## API Route Implementation

Create `src/pages/api/github-content.ts`:

```
import { Octokit } from "@octokit/rest";
import type { APIRoute } from "astro";

const octokit = new Octokit({
  auth: import.meta.env.GITHUB_TOKEN
});

export const get: APIRoute = async ({ params, request }) => {
  const url = new URL(request.url);
  const owner = url.searchParams.get('owner');
  const repo = url.searchParams.get('repo');
  const path = url.searchParams.get('path');

  if (!owner || !repo || !path) {
    return new Response(JSON.stringify({ error: 'Missing parameters' }), { status: 400 });
  }

  try {
    const response = await octokit.repos.getContent({
      owner,
      repo,
      path,
    });

    if ('content' in response.data) {
      const content = Buffer.from(response.data.content, 'base64').toString();
      return new Response(JSON.stringify({ content }), { status: 200 });
    } else {
      return new Response(JSON.stringify({ error: 'File not found' }), { status: 404 });
    }
  } catch (error) {
    return new Response(JSON.stringify({ error: 'Error fetching content' }), { status: 500 });
  }
};
```

## React Component

Create a React component to fetch and display the content:

```
import React, { useState, useEffect } from 'react';

export default function GitHubContent() {
  const [content, setContent] = useState('');

  useEffect(() => {
    fetch('/api/github-content?owner=username&repo=repo-name&path=path/to/file.md')
      .then(res => res.json())
      .then(data => setContent(data.content));
  }, []);

  return ;
}
```

## Integrating in Astro

Use the React component in an Astro page:

```
---
import GitHubContent from '../components/GitHubContent.jsx';
---


  
    GitHub Content
  
  
    Content from GitHub
    
  

```

## Configuration

1. Set up a GitHub Personal Access Token:
   - Create a token with `repo` scope.
   - Add it to your Vercel project's environment variables as `GITHUB_TOKEN`.

2. Update your `astro.config.mjs`:

```
import { defineConfig } from 'astro/config';
import vercel from '@astrojs/vercel/serverless';
import react from "@astrojs/react";

export default defineConfig({
  output: 'server',
  adapter: vercel(),
  integrations: [react()],
});
```

This setup allows you to fetch content from a separate GitHub repository into your Vercel-deployed Astro + React project, enabling dynamic content updates without redeploying your main project.
```

This Markdown file provides a comprehensive guide on implementing GitHub content fetching in an Astro + React project deployed on Vercel.
