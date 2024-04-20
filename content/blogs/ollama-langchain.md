---
title: "Exploring the Power of Ollama with LangChainGo: Building AI Applications in Go"
date: 2024-04-19T00:00:00-07:00
draft: false
github_link: "https://github.com/bindrad/bindrad.github.io"
author: "Dhruv Bindra"
tags:
  - Ollama
  - Langchaingo
  - gemma
  - lamma3
image: /images/langchaingo-ollama.jpeg
description: ""
toc: 
---

Integrating Ollama with LangChainGo opens up exciting possibilities for developers looking to harness the power of AI in their Go projects. Ollama, a language model developed by Gemma, offers advanced natural language processing capabilities, while LangChainGo provides seamless integration with Go, allowing developers to build production-ready AI applications with ease.

Here's a closer look at how you can leverage Ollama with LangChainGo to create sophisticated AI applications:
```
package main

import (
	"context"
	"flag"
	"fmt"
	"log"

	"github.com/tmc/langchaingo/llms"
	"github.com/tmc/langchaingo/llms/ollama"
)

func main() {
	modelName := flag.String("model", "", "ollama model name")
	query := flag.String("query", "", "query to the model")
	stream := flag.Bool("stream", false, "stream output")
	flag.Parse()

	llm, err := ollama.New(ollama.WithModel(*modelName))
	if err != nil {
		log.Fatal(err)
	}

	ctx := context.Background()
	if !*stream {
		completion, err := llms.GenerateFromSinglePrompt(ctx, llm, *query)
		if err != nil {
			log.Fatal(err)
		}

		fmt.Println("Response:\n", completion)
	} else {
		_, err = llms.GenerateFromSinglePrompt(ctx, llm, *query,
			llms.WithStreamingFunc(func(ctx context.Context, chunk []byte) error {
				fmt.Printf("chunk len=%d: %s\n", len(chunk), chunk)
				return nil
			}))
		if err != nil {
			log.Fatal(err)
		}
	}

}
```
This code snippet demonstrates how to utilize Ollama with LangChainGo to generate responses from a given prompt. With Ollama's powerful language modeling capabilities and LangChainGo's seamless integration, developers can easily incorporate AI-driven features into their Go applications.

An interesting aspect to note is that while LangChainGo offers a convenient API standardized across LLM providers, its use is by no means required for this sample. Ollama itself provides a Go API as part of its structure, allowing developers to use it externally as well.

Whether you're building chatbots, language translation tools, or text analysis systems, Ollama and LangChainGo provide the tools and resources necessary to create sophisticated AI solutions in Go.

In conclusion, the combination of Ollama and LangChainGo represents a significant advancement in the realm of AI development in Go. By leveraging these technologies, developers can unlock new opportunities for innovation and create AI-powered applications that meet the demands of today's dynamic digital landscape.

The complete code is [available at GitHub](https://github.com/bindrad/using-ollama-with-langchaingo).
