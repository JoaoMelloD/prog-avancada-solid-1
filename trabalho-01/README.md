# Trabalho 01 - Refatoracao SOLID

## Codigo refatorado completo

```typescript
interface EstrategiaDesconto {
    calcular(valorTotal: number): number;
}

class DescontoClienteComum implements EstrategiaDesconto {
    calcular(valorTotal: number): number {
        return 0;
    }
}

class DescontoClienteVip implements EstrategiaDesconto {
    calcular(valorTotal: number): number {
        return valorTotal * 0.20;
    }
}

class DescontoClienteEstudante implements EstrategiaDesconto {
    calcular(valorTotal: number): number {
        return valorTotal * 0.10;
    }
}

class DescontoClientePremium implements EstrategiaDesconto {
    calcular(valorTotal: number): number {
        return valorTotal * 0.15;
    }
}

abstract class Pedido {
    constructor(
        private readonly id: string,
        private readonly valorTotal: number,
        private readonly estrategiaDesconto: EstrategiaDesconto
    ) {}

    obterId(): string {
        return this.id;
    }

    obterValorTotal(): number {
        return this.valorTotal;
    }

    calcularDesconto(): number {
        return this.estrategiaDesconto.calcular(this.valorTotal);
    }

    calcularSubtotalComDesconto(): number {
        return this.valorTotal - this.calcularDesconto();
    }

    abstract calcularTotal(): number;
}

class PedidoFisico extends Pedido {
    constructor(
        id: string,
        valorTotal: number,
        estrategiaDesconto: EstrategiaDesconto,
        private readonly valorFrete: number
    ) {
        super(id, valorTotal, estrategiaDesconto);
    }

    calcularFrete(): number {
        return this.valorFrete;
    }

    calcularTotal(): number {
        return this.calcularSubtotalComDesconto() + this.calcularFrete();
    }
}

class PedidoDigital extends Pedido {
    calcularTotal(): number {
        return this.calcularSubtotalComDesconto();
    }
}

interface RepositorioPedido {
    salvar(pedido: Pedido): void;
}

class RepositorioPedidoMySQL implements RepositorioPedido {
    salvar(pedido: Pedido): void {
        console.log(`Salvando pedido ${pedido.obterId()} no MySQL.`);
    }
}

interface NotificadorPedido {
    enviarConfirmacao(pedido: Pedido): void;
}

class ServicoDeEmailConfirmacao implements NotificadorPedido {
    enviarConfirmacao(pedido: Pedido): void {
        console.log(`Enviando e-mail de confirmacao do pedido ${pedido.obterId()}.`);
    }
}

interface ProcessadorPagamento {
    processar(pedido: Pedido): void;
}

class ProcessadorPagamentoOnline implements ProcessadorPagamento {
    processar(pedido: Pedido): void {
        console.log(`Pagamento online processado para o pedido ${pedido.obterId()}.`);
    }
}

interface GeradorNotaFiscal {
    gerar(pedido: Pedido): void;
}

class GeradorNotaFiscalDigital implements GeradorNotaFiscal {
    gerar(pedido: Pedido): void {
        console.log(`Nota fiscal digital gerada para o pedido ${pedido.obterId()}.`);
    }
}

class GeradorNotaFiscalFisica implements GeradorNotaFiscal {
    gerar(pedido: Pedido): void {
        console.log(`Nota fiscal fisica gerada para o pedido ${pedido.obterId()}.`);
    }
}

interface ImpressorEtiquetaFisica {
    imprimir(pedido: PedidoFisico): void;
}

class ImpressoraEtiquetaCorreios implements ImpressorEtiquetaFisica {
    imprimir(pedido: PedidoFisico): void {
        console.log(`Etiqueta fisica impressa para o pedido ${pedido.obterId()}.`);
    }
}

class ServicoDePedido {
    constructor(
        private readonly repositorioPedido: RepositorioPedido,
        private readonly notificadorPedido: NotificadorPedido
    ) {}

    processarPedido(
        pedido: Pedido,
        processadorPagamento: ProcessadorPagamento,
        geradorNotaFiscal: GeradorNotaFiscal
    ): void {
        processadorPagamento.processar(pedido);
        geradorNotaFiscal.gerar(pedido);
        this.repositorioPedido.salvar(pedido);
        this.notificadorPedido.enviarConfirmacao(pedido);
    }

    processarPedidoFisico(
        pedido: PedidoFisico,
        processadorPagamento: ProcessadorPagamento,
        geradorNotaFiscal: GeradorNotaFiscal,
        impressorEtiquetaFisica: ImpressorEtiquetaFisica
    ): void {
        this.processarPedido(pedido, processadorPagamento, geradorNotaFiscal);
        impressorEtiquetaFisica.imprimir(pedido);
    }
}

const repositorioPedido = new RepositorioPedidoMySQL();
const servicoEmail = new ServicoDeEmailConfirmacao();
const servicoDePedido = new ServicoDePedido(repositorioPedido, servicoEmail);

const pedidoDigital = new PedidoDigital(
    "PED-001",
    200,
    new DescontoClienteVip()
);

const pedidoFisico = new PedidoFisico(
    "PED-002",
    350,
    new DescontoClientePremium(),
    15
);

servicoDePedido.processarPedido(
    pedidoDigital,
    new ProcessadorPagamentoOnline(),
    new GeradorNotaFiscalDigital()
);

servicoDePedido.processarPedidoFisico(
    pedidoFisico,
    new ProcessadorPagamentoOnline(),
    new GeradorNotaFiscalFisica(),
    new ImpressoraEtiquetaCorreios()
);
```

## Consideracoes sobre os principios SOLID

**SRP:** A classe `Pedido` deixou de concentrar responsabilidades de persistencia, envio de e-mail, pagamento e nota fiscal. Essas tarefas foram movidas para servicos e interfaces especificas, deixando o pedido responsavel apenas por representar os dados e calcular seus valores.

**OCP:** A regra de desconto passou a depender da interface `EstrategiaDesconto`. Para adicionar um novo tipo de cliente, como `DescontoClientePremium`, basta criar uma nova classe que implemente essa interface, sem alterar o codigo da classe `Pedido`.

**LSP:** `PedidoDigital` nao herda mais um comportamento de frete que nao consegue cumprir. O calculo de frete existe apenas em `PedidoFisico`, enquanto `PedidoDigital` calcula seu total sem precisar lancar excecoes.

**ISP:** A antiga interface generica foi separada em interfaces menores: `ProcessadorPagamento`, `GeradorNotaFiscal` e `ImpressorEtiquetaFisica`. Assim, pedidos digitais nao sao obrigados a implementar impressao de etiqueta fisica.

**DIP:** `ServicoDePedido` depende das abstracoes `RepositorioPedido` e `NotificadorPedido`, recebidas por injecao no construtor. Com isso, o sistema nao fica preso ao MySQL e pode trocar a persistencia ou criar testes usando implementacoes simuladas.
