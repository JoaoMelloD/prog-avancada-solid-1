Contexto do Problema:Você é o mais novo Arquiteto de Software da startup "OmniAI", uma plataforma que oferece serviços de Inteligência Artificial para empresas. A plataforma permite que os usuários gerem textos, imagens e áudios utilizando diferentes modelos de IA, tudo no mesmo lugar.

O MVP (Produto Mínimo Viável) foi um sucesso, mas o código inicial foi feito às pressas. Agora, a empresa precisa adicionar novos recursos (como geração de vídeo e troca do sistema de pagamentos), mas os desenvolvedores estão com medo de alterar o código atual porque ele quebra com facilidade.

Código Legado (TypeScript):

TypeScript
// 1. Sistema de cobrança engessado
class SistemaCobrancaStripe {
    cobrar(usuarioId: string, valorTokens: number): void {
        console.log(`Cobrando R$${valorTokens} via Stripe do usuário ${usuarioId}`);
    }
}

// 2. Interface "Faz-Tudo"
interface IModelosIA {
    gerarTexto(prompt: string): string;
    gerarImagem(prompt: string): string;
    gerarAudio(prompt: string): string;
}

// 3. A classe principal que gerencia tudo
class AssistenteOmniIA implements IModelosIA {
    public nomeModelo: string;

    constructor(nomeModelo: string) {
        this.nomeModelo = nomeModelo;
    }

    // Processador central cheio de condicionais
    processarRequisicaoUsuario(prompt: string, tipo: string): void {
        console.log(`Iniciando processamento com ${this.nomeModelo}...`);

        if (tipo === "TEXTO") {
            this.gerarTexto(prompt);
        } else if (tipo === "IMAGEM") {
            this.gerarImagem(prompt);
        } else if (tipo === "AUDIO") {
            this.gerarAudio(prompt);
        } else {
            throw new Error("Tipo de IA não suportado pelo sistema.");
        }
       
        // Finaliza cobrando o usuário direto aqui
        this.registrarCobranca(1.50);
    }

    gerarTexto(prompt: string): string {
        return `[Texto Gerado]: Respondendo ao prompt: ${prompt}`;
    }

    gerarImagem(prompt: string): string {
        return `[Imagem Gerada]: URL da imagem baseada em: ${prompt}`;
    }

    gerarAudio(prompt: string): string {
        return `[Áudio Gerado]: Arquivo de voz para: ${prompt}`;
    }

    registrarCobranca(valor: number): void {
        const stripe = new SistemaCobrancaStripe();
        stripe.cobrar("user_999", valor);
    }
}

// 4. Um modelo específico sendo forçado a herdar o que não deve
class ModeloFocadoEmTexto extends AssistenteOmniIA {
    constructor() {
        super("ChatGPT-4");
    }

    gerarImagem(prompt: string): string {
        throw new Error("Falha Crítica: O ChatGPT-4 não gera imagens nativamente nesta versão.");
    }

    gerarAudio(prompt: string): string {
        throw new Error("Falha Crítica: Modelo de texto não pode gerar arquivos de áudio.");
    }
}

Missão:O sistema está travando o crescimento da startup. Sua missão é reescrever a arquitetura desse sistema aplicando os 5 princípios S.O.L.I.D.

Requisitos da Refatoração:
SRP (Single Responsibility Principle): A classe AssistenteOmniIA está atuando como roteador de requisições, geradora de conteúdo e faturadora. Separe o serviço de cobrança da lógica de geração de IA.

OCP (Open/Closed Principle): O método processarRequisicaoUsuario usa if/else para decidir o que fazer. Se a empresa quiser lançar a geração de Vídeos amanhã, você terá que alterar essa classe. Refatore usando interfaces ou classes abstratas para que novos tipos de IA sejam adicionados sem tocar no código existente.

LSP (Liskov Substitution Principle): A classe ModeloFocadoEmTexto herda de AssistenteOmniIA, mas lança erros ao tentar gerar imagens e áudios. Isso quebra a aplicação caso alguém espere o comportamento da classe pai. Corrija a hierarquia para que subclasses nunca quebrem os contratos esperados.

ISP (Interface Segregation Principle): A interface IModelosIA obriga qualquer IA a saber gerar texto, imagem e áudio. Divida essa interface em partes menores e mais coesas (ex: geradores apenas de texto).

DIP (Dependency Inversion Principle): A cobrança no método registrarCobranca está fortemente acoplada ao SistemaCobrancaStripe. Se a startup quiser mudar para o PayPal ou Pix, terá muito trabalho. Faça o sistema depender de uma abstração (interface de pagamento) em vez da implementação concreta do Stripe.

Formato da Entrega: Apresente o código refatorado completo e adicione breves comentários (ou um parágrafo explicativo ao final) justificando as mudanças arquiteturais que você tomou para atender a cada uma das 5 questões acima. O código deverá ser entregue através do github. Cada um será avaliado pela porcentagem de commits, tomem cuidado !!