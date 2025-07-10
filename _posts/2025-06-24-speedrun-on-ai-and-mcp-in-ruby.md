---
excerpt: Model Context Protocol (MCP) talk at Ruby Melbourne
collection: learning
categories: 
tags:
  - software_engineering
  - learning
  - ai
  - "#mcp"
toc: true
toc_sticky: true
title: "Ruby Melbourne talk: speed-run on AI and MCP with Ruby"
permalink: speedrun-on-ai-and-mcp-in-ruby
---
---

# Action Plan

```ruby
1. Side Intro
	Prompt Engineering
		- temperature, top-P, top-K
	Retrieval-Augmented Generation (RAG)
2. ModelContextProtocol (MCP)
- live 'vibe' code an app and integrate with LLM client
- have some fun
```

---
# About me

```ruby
class Friendlyantz
  def initialize
    @name = 'Anton'
    @title = 'Extreme Programmer', 'Software Stuntman'
    @hobbies = ['kitesurfing', 'skatebords', 'MTB']
  end
  
  def find_more = 'https://friendlyantz.me/'
```

---

# Intro: Prompt Engineering
```ruby
class Intro::PromptEngineering
	temperature
	top-K
	top-P
end
```

---

![temp topp topK](https://github.com/user-attachments/assets/c83ab110-2414-42c1-afe8-01cbf5ba246b)

---

## Temperate (0…1)
- `0` - predictable, ‘dry’, sober
- `1` - most creative

 > Start with 0.2

---

## Top K(1…) 

max count of words with the highest probabilities. Start with 30 
```js
- I.e. `topK = 5`
["wine" (30%), "vino" (25%), "VB" (20%), "vermouth" (15%), "vine" (10%)]
```

---
## Top P(0..1) 

The model sorts all words by probability. 
It keeps adding words until their cumulative probability exceeds P. good start with 0.95 
```js
		(i.e. 0.9) -> 

["Wine" (20%), "Vino" (20%), "Vinho" (20%), "Wein" (20%), "GrapeJuice" (10%), ...]
```


---

> Note: Extreme values cancel each other out

---
```ruby
"Balanced / coherent results with a touch of creativity"
	- temperature of .2,
	- top-P of .95, 
	- top-K of 30 
"Especially creative results"
	- temperature of .9
	- top-P of .99
	- top-K of 40. 
"Less creative results"
	- temperature of .1
	- top-P of .9
	- top-K of 20.
```

---

# Retrieval-augmented generation (RAG)

---

![Pasted image 20250624171218](https://github.com/user-attachments/assets/b6d40ea0-9627-46ea-9707-0a311c08bcb2)


---

![rag-queen](https://github.com/user-attachments/assets/b3c8be5b-4933-43a9-8980-ac2dc753de61)

---

RAG / embeddings are not the final answer, other resources are available in real world

---
# Model Context Protocol (MCP) 

---

![Pasted image 20250624171847](https://github.com/user-attachments/assets/8c87292b-4f18-408f-9c6d-a4744fdaa3ba)


---

great explanation of MCP
[https://www.youtube.com/watch?v=FLpS7OfD5-s](https://www.youtube.com/watch?v=FLpS7OfD5-s)

![Pasted image 20250624173333](https://github.com/user-attachments/assets/d0526fff-5413-450c-8e04-dab7da9a9750)
---

[https://github.com/yjacquin/fast-mcp](https://github.com/yjacquin/fast-mcp)

---

# Time for...

---

# Vibe coding!

---

```
create a simple shopping app that lists some funny vibing products in valid JSON , include 5 example products. make it simple, no need in web server, sinatra, etc, just simple calss. just vibes class. name file `vibes.rb`
```

```
add shopping cart functionality that can take quantity
```

---

```ruby
require "json"

class Vibes
  def initialize
    @products = [
      {id: 1, name: "Dancing Cactus", price: 19.99, description: "A cactus that dances to your favorite tunes."},
      {id: 2, name: "Vibing Sloth Mug", price: 12.99, description: "A mug with a sloth that vibes with your coffee."},
      {id: 3, name: "Mood Ring Lamp", price: 29.99, description: "A lamp that changes colors based on your mood."},
      {id: 4, name: "Chill Llama Pillow", price: 24.99, description: "A pillow shaped like a llama for ultimate chill vibes."},
      {id: 5, name: "Groovy Disco Ball", price: 34.99, description: "A mini disco ball to bring the groove to your room."}
    ]
    @cart = []
  end

  def list_products
    JSON.pretty_generate({products: @products})
  end

  def add_to_cart(product_id, quantity)
    product = @products.find { |p| p[:id] == product_id }
    if product
      @cart << {product: product, quantity: quantity}
      "Added #{quantity} of '#{product[:name]}' to the cart."
    else
      "Product not found."
    end
  end

  def view_cart
    JSON.pretty_generate(@cart.map do |item|
      {
        name: item[:product][:name],
        quantity: item[:quantity],
        total_price: (item[:product][:price] * item[:quantity]).round(2)
      }
    end)
  end
end

VIBES = Vibes.new

require "fast_mcp"

server = FastMcp::Server.new(name: "my-ai-server", version: "1.0.0")

class VibeProducts < FastMcp::Tool
  description "List some funny vibing products"

  def call
    VIBES.list_products
  end
end
server.register_tool(VibeProducts)

class AddToCart < FastMcp::Tool
  description "Add a product to the cart"

  arguments do
    required(:product_id).filled(:integer).description("The ID of the product to add")
    required(:quantity).filled(:integer).description("The quantity of the product to add")
  end

  def call(product_id:, quantity:)
    VIBES.add_to_cart(product_id.to_i, quantity.to_i)
  end
end
server.register_tool(AddToCart)

class ViewCart < FastMcp::Tool
  description "View the current shopping cart"

  def call
    VIBES.view_cart
  end
end

server.register_tool(ViewCart)
server.start
```
---

Claude Desktop agent `claude_desktop_config.json`

```json
{
  "mcpServers": {
    "vibe-mcp": {
      "command": "/PATH/ruby/3.4.4/bin/ruby",
      "args": [
        "/PATH/mcp-vibe/vibes.rb"
      ]
    },
    "mp-challenge": {
      "command": "/PATH/ruby/3.4.4/bin/ruby",
      "args": [
        "/PATH/promotions/app/app.rb"
      ]
    }
  }
}
```

chat history
[https://claude.ai/share/69c174c5-40b1-4f24-9769-583541d71c95
](https://claude.ai/share/69c174c5-40b1-4f24-9769-583541d71c95
)
---
REFERENCES:
- [Prompt engineering paper](https://drive.google.com/file/d/1AbaBYbEa_EbPelsT40-vj64L-2IwUJHy/view) by google
- [https://docs.anthropic.com/en/docs/mcp](https://docs.anthropic.com/en/docs/mcp)
