# Trabalho 02 - Refatoracao SOLID OmniAI

## Codigo refatorado completo

```typescript
type ResultadoGeracao = {
    tipo: string;
    conteudo: string;
    valorCobranca: number;
};

interface GeradorIA {
    obterNomeModelo(): string;
    gerar(prompt: string): ResultadoGeracao;
}

interface GeradorDeTexto {
    gerarTexto(prompt: string): string;
}

interface GeradorDeImagem {
    gerarImagem(prompt: string): string;
}

interface GeradorDeAudio {
    gerarAudio(prompt: string): string;
}

interface GeradorDeVideo {
    gerarVideo(prompt: string): string;
}

class GeradorTexto implements GeradorIA, GeradorDeTexto {
    constructor(private readonly nomeModelo: string) {}

    obterNomeModelo(): string {
        return this.nomeModelo;
    }

    gerarTexto(prompt: string): string {
        return `[Texto Gerado]: Respondendo ao prompt: ${prompt}`;
    }

    gerar(prompt: string): ResultadoGeracao {
        return {
            tipo: "TEXTO",
            conteudo: this.gerarTexto(prompt),
            valorCobranca: 1.50
        };
    }
}

class GeradorImagem implements GeradorIA, GeradorDeImagem {
    constructor(private readonly nomeModelo: string) {}

    obterNomeModelo(): string {
        return this.nomeModelo;
    }

    gerarImagem(prompt: string): string {
        return `[Imagem Gerada]: URL da imagem baseada em: ${prompt}`;
    }

    gerar(prompt: string): ResultadoGeracao {
        return {
            tipo: "IMAGEM",
            conteudo: this.gerarImagem(prompt),
            valorCobranca: 2.50
        };
    }
}

class GeradorAudio implements GeradorIA, GeradorDeAudio {
    constructor(private readonly nomeModelo: string) {}

    obterNomeModelo(): string {
        return this.nomeModelo;
    }

    gerarAudio(prompt: string): string {
        return `[Audio Gerado]: Arquivo de voz para: ${prompt}`;
    }

    gerar(prompt: string): ResultadoGeracao {
        return {
            tipo: "AUDIO",
            conteudo: this.gerarAudio(prompt),
            valorCobranca: 3.00
        };
    }
}

class GeradorVideo implements GeradorIA, GeradorDeVideo {
    constructor(private readonly nomeModelo: string) {}

    obterNomeModelo(): string {
        return this.nomeModelo;
    }

    gerarVideo(prompt: string): string {
        return `[Video Gerado]: URL do video baseado em: ${prompt}`;
    }

    gerar(prompt: string): ResultadoGeracao {
        return {
            tipo: "VIDEO",
            conteudo: this.gerarVideo(prompt),
            valorCobranca: 6.00
        };
    }
}

class ModeloFocadoEmTexto extends GeradorTexto {
    constructor() {
        super("ChatGPT-4");
    }
}

interface GatewayPagamento {
    cobrar(usuarioId: string, valor: number): void;
}

class SistemaCobrancaStripe implements GatewayPagamento {
    cobrar(usuarioId: string, valor: number): void {
        console.log(`Cobrando R$${valor.toFixed(2)} via Stripe do usuario ${usuarioId}.`);
    }
}

class SistemaCobrancaPayPal implements GatewayPagamento {
    cobrar(usuarioId: string, valor: number): void {
        console.log(`Cobrando R$${valor.toFixed(2)} via PayPal do usuario ${usuarioId}.`);
    }
}

interface CobradorUso {
    cobrarUso(usuarioId: string, valor: number): void;
}

class ServicoCobranca implements CobradorUso {
    constructor(private readonly gatewayPagamento: GatewayPagamento) {}

    cobrarUso(usuarioId: string, valor: number): void {
        this.gatewayPagamento.cobrar(usuarioId, valor);
    }
}

class ServicoOmniAI {
    constructor(private readonly cobradorUso: CobradorUso) {}

    processarRequisicaoUsuario(
        usuarioId: string,
        prompt: string,
        gerador: GeradorIA
    ): ResultadoGeracao {
        console.log(`Iniciando processamento com ${gerador.obterNomeModelo()}...`);

        const resultado = gerador.gerar(prompt);
        this.cobradorUso.cobrarUso(usuarioId, resultado.valorCobranca);

        return resultado;
    }
}

const gatewayPagamento = new SistemaCobrancaStripe();
const servicoCobranca = new ServicoCobranca(gatewayPagamento);
const servicoOmniAI = new ServicoOmniAI(servicoCobranca);

const geradorTexto = new ModeloFocadoEmTexto();
const geradorImagem = new GeradorImagem("DALL-E");
const geradorAudio = new GeradorAudio("VoiceAI");
const geradorVideo = new GeradorVideo("VideoGen");

const resultadoTexto = servicoOmniAI.processarRequisicaoUsuario(
    "user_999",
    "Explique SOLID em poucas palavras",
    geradorTexto
);

const resultadoImagem = servicoOmniAI.processarRequisicaoUsuario(
    "user_999",
    "Uma cidade futurista ao entardecer",
    geradorImagem
);

const resultadoAudio = servicoOmniAI.processarRequisicaoUsuario(
    "user_999",
    "Narracao profissional para um anuncio",
    geradorAudio
);

const resultadoVideo = servicoOmniAI.processarRequisicaoUsuario(
    "user_999",
    "Apresentacao curta de uma plataforma de IA",
    geradorVideo
);

console.log(resultadoTexto.conteudo);
console.log(resultadoImagem.conteudo);
console.log(resultadoAudio.conteudo);
console.log(resultadoVideo.conteudo);
```

## Consideracoes sobre os principios SOLID

**SRP:** A responsabilidade de gerar conteudo ficou nas classes de geracao, a cobranca ficou em `ServicoCobranca` e a orquestracao ficou em `ServicoOmniAI`. Assim, cada classe tem um motivo principal para mudar.

**OCP:** `ServicoOmniAI` nao possui condicionais para decidir entre texto, imagem, audio ou video. Para adicionar um novo tipo de IA, basta criar uma nova classe que implemente `GeradorIA`, sem alterar o servico principal.

**LSP:** `ModeloFocadoEmTexto` nao herda mais metodos de imagem ou audio que nao consegue cumprir. Ele deriva de `GeradorTexto` e respeita todos os contratos que expoe, sem lancar erros por funcionalidades incompativeis.

**ISP:** A interface faz-tudo foi dividida em contratos menores: `GeradorDeTexto`, `GeradorDeImagem`, `GeradorDeAudio` e `GeradorDeVideo`. Cada modelo implementa apenas as capacidades que realmente possui.

**DIP:** A cobranca depende da abstracao `GatewayPagamento`, recebida por injecao em `ServicoCobranca`. Desse modo, trocar Stripe por PayPal, Pix ou outro provedor exige apenas outra implementacao da interface, sem alterar a regra de negocio.
