Contexto do Problema:Você foi contratado(a) como Desenvolvedor(a) para modernizar o sistema de um E-commerce em crescimento. Ao analisar o repositório, você encontrou o código base responsável por gerenciar os pedidos da loja.

O desenvolvedor anterior centralizou quase toda a regra de negócios em poucas estruturas, resultando em um código fortemente acoplado, rígido e propenso a bugs durante manutenções.

Código Legado (TypeScript):

TypeScript
// 1. Classe de Banco de Dados Concreta
class BancoDeDadosMySQL {
    salvar(dados: any): void {
        console.log("Salvando dados no MySQL...");
    }
}

// 2. Interface de tarefas do pedido
interface ITarefasPedido {
    processarPagamento(): void;
    gerarNotaFiscal(): void;
    imprimirEtiquetaFisica(): void;
}

// 3. Classe principal de Pedido
class Pedido {
    public valorTotal: number;
    public tipoCliente: string;

    constructor(valorTotal: number, tipoCliente: string) {
        this.valorTotal = valorTotal;
        this.tipoCliente = tipoCliente;
    }

    calcularDesconto(): number {
        if (this.tipoCliente === "VIP") {
            return this.valorTotal * 0.20;
        } else if (this.tipoCliente === "ESTUDANTE") {
            return this.valorTotal * 0.10;
        }
        return 0;
    }

    calcularFrete(): number {
        return 15.0;
    }

    salvarPedido(): void {
        const db = new BancoDeDadosMySQL();
        db.salvar(this);
    }

    enviarEmailConfirmacao(): void {
        console.log("Enviando e-mail de confirmação para o cliente...");
    }
}

// 4. Implementação para produtos digitais
class PedidoProdutoDigital extends Pedido implements ITarefasPedido {
   
    calcularFrete(): number {
        throw new Error("Erro: Produtos digitais não possuem frete.");
    }

    processarPagamento(): void {
        console.log("Pagamento processado online.");
    }

    gerarNotaFiscal(): void {
        console.log("Nota fiscal digital gerada.");
    }

    imprimirEtiquetaFisica(): void {
        throw new Error("Erro: Não é possível imprimir etiqueta para produto digital.");
    }
}


Missão:O código acima viola frontalmente os cinco princípios do S.O.L.I.D. Sua tarefa é refatorar o código, garantindo que ele seja escalável, testável e de fácil manutenção.

Requisitos da Entrega:
1. Single Responsibility Principle (SRP)
A classe Pedido possui múltiplas razões para mudar (cálculos, persistência no banco, envio de e-mails). Refatore o código extraindo essas responsabilidades para classes ou serviços específicos.

2. Open/Closed Principle (OCP)
A adição de um novo tipo de cliente (ex: "PREMIUM") exigiria a modificação direta do método calcularDesconto(), o que é uma má prática. Altere a estrutura de descontos para que o sistema fique aberto para extensão, mas fechado para modificação (dica: utilize polimorfismo/interfaces).

3. Liskov Substitution Principle (LSP)
A classe PedidoProdutoDigital lança uma exceção no método calcularFrete(), quebrando a confiança de quem consome a classe base Pedido. Reestruture a hierarquia de classes ou o uso de interfaces para que as classes filhas possam substituir a classe pai sem quebrar o sistema.

4. Interface Segregation Principle (ISP)
A interface ITarefasPedido é muito genérica e força a classe PedidoProdutoDigital a implementar o método imprimirEtiquetaFisica(), lançando um erro. Segregue esta interface em interfaces menores e mais coesas.

5. Dependency Inversion Principle (DIP)
O método salvarPedido() instancia diretamente a classe concreta BancoDeDadosMySQL. Isso impede que o sistema mude de banco de dados no futuro ou que testes unitários sejam feitos facilmente. Aplique a inversão de dependência fazendo com que a persistência dependa de uma abstração e seja injetada no sistema.

Formato da Entrega:Apresente o código refatorado completo e adicione breves comentários (ou um parágrafo explicativo ao final) justificando as mudanças arquiteturais que você tomou para atender a cada uma das 5 questões acima. O código deverá ser entregue através do github. Cada um será avaliado pela porcentagem de commits, tomem cuidado !!