---
title: Escrevendo Front End em  Rust
date: 2023-12-19
---

# Inicio

Esse fim de Semana decidi me aventurar em algo diferente, eu nesse momento estou aprendendo em Rust e dando uma olhada em outras linguagens que são considerada destinadas a "low level", e acabei descobrindo que existe um framework chamado "Yew" pra Rust que possibilita frontend ser feito com Rust. Logo pensei "hum... isso parece legal e eu tenho que dar uma olhada em frontend vou tentar". Então me deparei com uma estrutura semelhantes a frameworks modernos de Javascript, mas que também me possibilita fazer algo como se fosse apenas html, css e JS, e como minha mãe dizia:
` - Mente vazia, oficina do Diabo.`
No entanto, decidi fazer o projeto que todo programador sênior tem em seu portfólio: `A LENDÁRIA POKEDEX.`

#  "Arquitetura", se é que se pode chamar assim.

Basicamente é como se fosse um aplicativo com html, css e js. Porém com a adição da pasta favicon para o favicon (ícone do aplicativo).

# Tecnologias usadas

[HTML](https://developer.mozilla.org/en-US/docs/Web/HTML) - Apesar de tudo, o HTML não é muito usado para nada além de funções de link.


[CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) - Estiliza o código assim como se fosse normalmente.


[WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly) - Webassembly faz boa parte da "magica" do nosso aplicaitvo, fazendo desde de a interface entre Rust, browser e JS até o nosso server.


[Trunk](https://trunkrs.dev/) - Trunk é um bundler para Webassembly com Rust, na nossa aplicação é vital para rodar o server e renderizar elementos do nosso HTML.


[Rust](https://doc.rust-lang.org/book/) - Foi a linguagem escolhida, apesar de boa parte das funções serem adeptas do JS e virarem JS no final, o código ainda é bem no "estilo Rust" e usa bastante coisa da linguagem.

# Como tudo começou?

Bom, estava eu conversando com um amigo  sobre tecnologias web e me veio a ideia de procurar algo diferente pra fazer frontend e logo pensei "com certeza deve ter algo em Rust" e acabou que eu achei o Yew que é um framework de Rust para frontend, ele tem suporte pra maioria das tecnologias da web, pelo o que vi antigamente o Yew fazia bem mais coisa por si mesmo, mas agora ele foca mais na componentização e deixa mais coisas pro WASM.
Tecnologia escolhida, certo? Hora de fazer algum projeto. Eu desde do inicio não queria nada complexo, só queria testar o framework e ver como me sairia então logo pensei em algo fácil de achar e que provavelmente é um "must" para quem está aprendendo: Pokedex, logo achei conteúdo bem fácil e didático (Credito ao [Manual do Dev](https://www.youtube.com/watch?v=SjtdH3dWLa8)) então peguei a ideia do design e como o Javascript se integraria nisso.

# Primeiras passos no Yew

Como não era nada difícil, uma lida bem rápida na documentação já é até oferecido um template de app mais simples que já iria me servir, já que eu queria fazer algo simples que é só praticamente uma página. Então só tive que dar uma olhada rápida e a macro `html![]` ajudou bastante já que é praticamente escrever html normal, com a diferença que qualquer coisa que  venha do Rust tem que ser passado entre chaves `{}`, se você quer passar um texto no elemento html vai ter que fazer com chaves, por exemplo:
```Rust 
use yew::prelude::*;

#[function_component(App)]
fn app() -> Html { 
    html![
        <h1>{"Olá, mundo!"}</h1>
    ]
}
```
Caso o que você quer passar pertença ao html pode passar como string mesmo:
```Rust
use yew::prelude::*;

#[function_component(App)]
fn app() -> Html { 
    html![
        <h1 class="exemplo">{"Olá, mundo!"}</h1>
    ]
}
```

Até aqui estava tudo bem, mas quando fui rodar o projeto, percebi que algumas coisas não apareciam e logo fui pesquisar, descobri que para renderizar as coisas do html que vem direto do html mesmo, precisamos usar a tag do html `data-trunk` que tem sua própria syntax, por exemplo:
```html
Por exemplo no html normalmente seria dessa forma
<link rel="stylesheet" href="path">
Mas com o trunk é gerado dessa forma
<link data-trunk rel="css" href="./css/style.css">
```

Até aqui nenhum desafio e até aí... eu estava achando fácil e que não teria maiores problemas, mas eu não sabia o que estava por vir.

# Javascript... o problema

Bom... até agora estava tudo indo bem até que veio o Javascript acabar com tudo isso, exemplificando, se eu estivesse no Javascript, normalmente, eu poderia usar [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) para consumir a pokeapi, mas como estou no Rust/Webassembly, não é tão simples assim e aqui começa a complicar, já que a documentação não é tão clara sobre e não tem muito material na internet, até agora já se passou um dia 1 e meio (comecei na madrugada de sabado) e nesse momento eu já estava tentando de tudo, conseguindo até alguns resultados, mas nada satisfatório e cheio de erros.
No entanto, eu pensei numa máxima da programação "Não existe um problema que ninguém já não tenha passado" e fui buscar por conteúdo de Rust com a pokeapi e descobri que até existe uma crate (lib), mas até aí nada muito útil pra mim. Até que eu achei esse repositório [Obrigado, amigo, me ajudou muito](https://github.com/apinanyogaratnam/yew-whos-that-pokemon-client/blob/main/src/main.rs) e me ajudou bastante na lógica e incorporei algumas coisas desse codigo no meu (bastante até pra ser sincero), mas muita coisa que ele faz é diferente então acabou que muita coisa não funcionou.

# Closures e seu escopo

Após completar 2 dias decidi ir descansar com alguns problemas ainda, bom especificamente um problema com closures que as variáveis iam pra fora do escopo da closure... e bom, como eu já estava descansado, eu pensei o seguinte "Ah só fazer uma função" e realmente até aí resolveu, mas gerou MAIS UM PROBLEMA, mas esse problema daí já tinha resolvido e foi um alívio. Só pra contexto, o código tava assim, não exatamente, mas algumas coisas estavam dessa forma, mas vai ilustrar meu ponto:

```Rust
use std::ops::Add;
use yew::prelude::*;
use web_sys::HtmlInputElement;
use serde_json::Value;

#[function_component(App)]
fn app() -> Html {
    let pokemon_state = use_state_eq::<Option<Pokemon>, _>(|| None);
    let search_state = use_state_eq::<i32, _>(|| 1.max(1));
    let search_state_outer = search_state.clone();
    let pokemon_state_outer = pokemon_state.clone();
    let onclick_prev: Callback<MouseEvent> = Callback::from(move |mouse_event| {
        on_click(search_state_outer.clone(), pokemon_state_outer.clone(), _search_state.clone(), -1, mouse_event);
    });
    let onclick_next: Callback<MouseEvent> = Callback::from(move |mouse_event| {
        on_click(search_state_outer.clone(), pokemon_state_outer.clone(), _search_state.clone(), 1, mouse_event);
    });
    let onkeydown: Callback<KeyboardEvent> = Callback::from(move |keyboard_event: KeyboardEvent| {
        if keyboard_event.key() == "Enter" {
            keyboard_event.prevent_default();
            let input: HtmlInputElement = keyboard_event.target_unchecked_into();
            let value = input.value();
            if let Ok(number) = value.parse::<i32>() {
                _search_state.set(number);
            }
        }
    });

    html! {
        <main>
            <img src={format!("{}", pokemon_state.as_ref().map_or("", |p| &p.image_src))} alt="pokemon" class="poke__image" />

            <h1 class="pokemon__data">
                <span class="pokemon__number">{format!("{:?} - ", search_state.to_string().parse::<i32>().unwrap())}</span>
                <span class="pokemon__name">{format!("{}", pokemon_state.as_ref().map_or("", |p| &p.name))}</span>
            </h1>

            <form class="form">
                <input
                    type="search"
                    class="input__search"
                    placeholder="Name or number"
                    onkeydown={onkeydown}
                />
            </form>

            <div class="buttons">
                <button class="button btn-prev" onclick={onclick_prev}>{"Prev <"}</button>
                <button class="button btn-next" onclick={onclick_next}>{"Next >"}</button>
            </div>
            <img src="./images/pokedex.png" alt="pokedex" class="pokedex" />
        </main>
    }
}

#[derive(Debug, PartialEq, Clone)]
struct Pokemon {
    id: i32,
    name: String,
    image_src: String,
}

fn on_click(
    search_state_outer: UseStateHandle<i32>,
    pokemon_state: UseStateHandle<Option<Pokemon>>,
    search_state: UseStateHandle<i32>,
    increment: i32,
    _mouse_event: MouseEvent,
) {
    search_state.set(search_state_outer.add(increment));

    let pokemon_state = pokemon_state.clone();

    wasm_bindgen_futures::spawn_local(async move {
        let url = format!("https://pokeapi.co/api/v2/pokemon/{}", search_state_outer.to_string());
        let response = reqwest::get(url).await.unwrap();

        if response.status().is_success() {
            let text = response.text().await.unwrap();
            let v: Value = serde_json::from_str(&text).unwrap();
            let name = v["name"].as_str().unwrap();
            let id = v["id"].as_i64().unwrap() as i32;
            let image_src = v["sprites"]["versions"]["generation-v"]["black-white"]["animated"]["front_default"]
                .as_str()
                .unwrap();

            let pokemon = Pokemon {
                id,
                name: name.into(),
                image_src: image_src.into(),
            };
            pokemon_state.set(Some(pokemon));
        } else {
            pokemon_state.set(None);
        }
    });
}

fn main() {
    yew::Renderer::<App>::new().render();
}

```

# O Fim e a resolução

Bom, até aí, eu já tinha resolvido um problema, mas tinha sobrado mais um... que se você sabe um pouco de Rust notou no snippet anterior, mas vou mostrar e explicar (tentar):

```Rust
let pokemon_state = use_state_eq::<Option<Pokemon>, _>(|| None);
    let search_state = use_state_eq::<i32, _>(|| 1.max(1));
    let search_state_outer = search_state.clone();
    let pokemon_state_outer = pokemon_state.clone();
    let onclick_prev: Callback<MouseEvent> = Callback::from(move |mouse_event| {
        on_click(search_state_outer.clone(), pokemon_state_outer.clone(), _search_state.clone(), -1, mouse_event);
    });
    let onclick_next: Callback<MouseEvent> = Callback::from(move |mouse_event| {
        on_click(search_state_outer.clone(), pokemon_state_outer.clone(), _search_state.clone(), 1, mouse_event);
    });
    let onkeydown: Callback<KeyboardEvent> = Callback::from(move |keyboard_event: KeyboardEvent| {
        if keyboard_event.key() == "Enter" {
            keyboard_event.prevent_default();
            let input: HtmlInputElement = keyboard_event.target_unchecked_into();
            let value = input.value();
            if let Ok(number) = value.parse::<i32>() {
                _search_state.set(number);
            }
        }
    });

```

Nesse trecho de código, note que as variáveis são copiadas, mas sua ownership foi movida para closure e o que isso quer dizer? EXATO, ela está no escopo da closure e não pode ser chamada... mas graças a deus, uma pessoa me ajudou me avisando sobre isso já que ela passou pelo mesmo problema e me ajudou a resolver, o código acabou ficando um pouco repetitivo, mas ficou assim:
```Rust

fn app() -> Html {
    let pokemon_state = use_state_eq::<Option<Pokemon>, _>(|| None);
    let search_state = use_state_eq::<i32, _>(|| 1.max(1));
    let _search_state = search_state.clone();
    let search_state_outer = search_state.clone();
    let _pokemon_state_outer = pokemon_state.clone();
    let pokemon_state_outer = pokemon_state.clone();
    let onclick_prev: Callback<MouseEvent> = Callback::from(move |mouse_event| {
        on_click(search_state_outer.clone(), pokemon_state_outer.clone(), _search_state.clone(), -1, mouse_event);
    });
    let _search_state = search_state.clone();
    let search_state_outer = search_state.clone();
    let pokemon_state_outer = pokemon_state.clone();
    let onclick_next: Callback<MouseEvent> = Callback::from(move |mouse_event| {
        on_click(search_state_outer.clone(), pokemon_state_outer.clone(), _search_state.clone(), 1, mouse_event);
    });
    let _search_state = search_state.clone();
    let onkeydown: Callback<KeyboardEvent> = Callback::from(move |keyboard_event: KeyboardEvent| {
        if keyboard_event.key() == "Enter" {
            keyboard_event.prevent_default();
            let input: HtmlInputElement = keyboard_event.target_unchecked_into();
            let value = input.value();
            if let Ok(number) = value.parse::<i32>() {
                _search_state.set(number);
            }
        }
    });
```

Bom, até aí resolvido, né? Só rodar e pronto... mas os POKÉMONS e as contagens estão desorganizadas! Droga... passei o dia inteiro e já estou morto. Acabei que não consegui resolver esses problemas e sinceramente, posso dizer que a melhor solução foi dar um tempo... com esse tempo dado consegui visualizar o problema de uma forma completamente diferente. Lembra que o Javascript até agora tinha sido um vilão? Então, nessa reta final ele se redimiu e virou o herói porque dando uma olhada no código acabei percebendo que a função para renderizar era desacoplada dos eventos e que a variável que devia estar sendo usada não era `search_state` e sim `pokemon_state` já que ambas funcionam diferente e em tempos diferentes, então é melhor usar pokemon_state para deixar tudo junto... e foi algo que passou completamente despercebido a princípio, mas vamos focar nas funções que foram o terror do dia, essa função:
```Rust 
fn on_click(
    search_state_outer: UseStateHandle<i32>,
    pokemon_state: UseStateHandle<Option<Pokemon>>,
    search_state: UseStateHandle<i32>,
    increment: i32,
    _mouse_event: MouseEvent,
) {
    search_state.set(search_state_outer.add(increment));

    let pokemon_state = pokemon_state.clone();

    wasm_bindgen_futures::spawn_local(async move {
        let url = format!("https://pokeapi.co/api/v2/pokemon/{}", search_state_outer.to_string());
        let response = reqwest::get(url).await.unwrap();

        if response.status().is_success() {
            let text = response.text().await.unwrap();
            let v: Value = serde_json::from_str(&text).unwrap();
            let name = v["name"].as_str().unwrap();
            let id = v["id"].as_i64().unwrap() as i32;
            let image_src = v["sprites"]["versions"]["generation-v"]["black-white"]["animated"]["front_default"]
                .as_str()
                .unwrap();

            let pokemon = Pokemon {
                id,
                name: name.into(),
                image_src: image_src.into(),
            };
            pokemon_state.set(Some(pokemon));
        } else {
            pokemon_state.set(None);
        }
    });
}
```

Virou essas duas:
```Rust
fn on_click(
    pokemon_state: UseStateHandle<Option<Pokemon>>,
    search_state: UseStateHandle<i32>,
    increment: i32,
    _mouse_event: MouseEvent,
) {
    search_state.set(search_state.add(increment));
    let search_state = search_state.clone();
    let pokemon_state = pokemon_state.clone();
    render_pokemon(pokemon_state, search_state)
}

fn render_pokemon(pokemon_state: UseStateHandle<Option<Pokemon>>, search_state: UseStateHandle<i32>) {
    wasm_bindgen_futures::spawn_local(async move {
        let url = format!("https://pokeapi.co/api/v2/pokemon/{}", search_state.to_string());
        let response = reqwest::get(url).await.unwrap();

        if response.status().is_success() {
            let text = response.text().await.unwrap();
            let v: Value = serde_json::from_str(&text).unwrap();
            let name = v["name"].as_str().unwrap();
            let id = v["id"].as_i64().unwrap() as i32;
            let image_src = v["sprites"]["versions"]["generation-v"]["black-white"]["animated"]["front_default"]
                .as_str()
                .unwrap();

            let pokemon = Pokemon {
                id,
                name: name.into(),
                image_src: image_src.into(),
            };
            pokemon_state.set(Some(pokemon));
        } else {
            pokemon_state.set(None);
        }
    });
}
```

Assim resolvendo o problema de incrementar errado e trazer os valores default para a pokedex, o código final ficou assim: [Achou que eu ia colocar mais um snippet, né?](https://github.com/Every2/Rust-pokedex)

# Considerações finais

Bom, eu acabei aprendendo bastante coisa sobre Rust e acabei gostando do Yew, pretendo fazer alguns projetos nele e talvez até mais complexos já que ele permite que bastante coisa possa ser feita nele. O código talvez tenha ficado bagunçado e vale a pena uma refatoração futura, mas eu acho que o resultado e o aprendizado valeram muito. Se você leu até aqui, Obrigado. :)
