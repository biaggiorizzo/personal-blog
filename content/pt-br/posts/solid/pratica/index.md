---
date: '2025-11-25T20:28:04-03:00'
title: 'Parte 2 - SOLID na Prática: Do Código Ruim ao Código Flexível'
translationKey: 
slug: "solid-na-pratica"
tags: ["poo", "arquitetura", "boas-praticas"]
cover:
    image: "images/roman-empire-battle-attack.webp"
    alt: "Ilustração representa a prática dos princípios SOLID" 
---

Na **Parte 2**, vamos transformar esses conceitos em código real.  
Para entender melhor os conceitos, recomendo ler antes a [**Parte 1**](/posts/solid/entendendo-principios-solid/), caso ainda não tenha visto.

A ideia é ir além da teoria e observar, diretamente no código:
- exemplos antes e depois (violação -> refatoração),
- diagramas de dependência aplicados ao código,
- como o SOLID melhora testabilidade, isolamento e clareza,
- e um exercício prático completo:  
    uma Central de Notificações com Email, SMS e Push, aplicando SRP, OCP e DIP de ponta a ponta.

A parte prática será demonstrada em **Python**, mas os princípios se aplicam a qualquer linguagem.  
A resolução final do exercício foi feita em **Java**, reforçando como o SOLID é universal, independentemente da sintaxe.  
O objetivo aqui é fechar o ciclo: entender como cada princípio se manifesta em decisões reais de design, não apenas no texto, mas em código que você pode testar, evoluir e refatorar.

**Pontos de atenção:**  
No Python, não existe a palavra-chave `interface` como em Java.  
Por isso, usamos o módulo `abc` (_Abstract Base Class_) e o decorator `@abstractmethod` para definir abstrações, contratos e polimorfismo seguro, permitindo aplicar OCP, LSP, ISP e DIP com precisão.


>[!TIP] Dica:
> Quando você sentir que uma mudança simples exige mexer em vários lugares, é um sinal de que um dos princípios SOLID foi violado.  
> O segredo não é decorar, mas sentir o incômodo e saber onde ajustar.

## **S — Single Responsibility Principle (Responsabilidade Única)**

> “Uma classe deve ter apenas um motivo para mudar.”

### Antes (violando o SRP)

{{< highlight python >}}
class SistemaPedido:
    def realizar_pedido(self, produto: str, quantidade: int):
        # validação
        if not isinstance(produto, str) or quantidade <= 0:
            print("Erro: dados inválidos.")
            return
        
        # cálculo do valor
        total = quantidade * 10  
        print(f"Pedido confirmado: {quantidade}x {produto} — total R${total}")
{{< /highlight >}}


**Problema:** Podemos observar que a classe `SistemaPedido` realiza a validação, calcula o preço e gera confirmação, possui um total de três responsabilidades.

### **Depois (respeitando SRP)**

{{< highlight python >}}
class PedidoValidador:
    def validar(self, produto: str, quantidade: int) -> bool:
        return isinstance(produto, str) and quantidade > 0


class PedidoPreco:
    def calcular(self, quantidade: int) -> float:
        return quantidade * 10  # mock de cálculo real


class PedidoConfirmacao:
    def enviar(self, produto: str, quantidade: int, total: float):
        print(f"Pedido confirmado: {quantidade}x {produto} — total R${total}")


class SistemaPedido:
    def __init__(self, validador: PedidoValidador, preco: PedidoPreco, confirmacao: PedidoConfirmacao):
        self.validador = validador
        self.preco = preco
        self.confirmacao = confirmacao

    def realizar_pedido(self, produto, quantidade):
        if not self.validador.validar(produto, quantidade):
            print("Erro: dados inválidos.")
            return
        
        total = self.preco.calcular(quantidade)
        self.confirmacao.enviar(produto, quantidade, total)
{{< /highlight >}}


| Antes                 | Depois                      |
| --------------------- | --------------------------- |
| Classe fazia tudo     | Cada uma tem papel definido |
| Mudanças quebram tudo | Mudanças isoladas           |
| Testes difíceis       | Testes simples e focados    |

**Resumo:**

SRP é sobre “motivos de mudança”.  
Se uma classe muda por mais de um motivo, ela tem responsabilidades demais.  
Separe em componentes pequenos, coesos e previsíveis.

## **O — Open/Closed Principle (Aberto/Fechado)**

> “Entidades de software devem estar abertas para extensão, mas fechadas para modificação.”

### Antes (violando o OCP)

{{< highlight python >}}
class EntregaService:
    def entregar(self, metodo):
        if metodo == "moto":
            print("Entrega via moto")
        elif metodo == "correios":
            print("Entrega via Correios")
        elif metodo == "drone":
            print("Entrega via drone")
{{< /highlight >}}


**Problema:** toda vez que entra um novo meio de entrega, exige modificar a classe `EntregaService`.  
Isso quebra o OCP e gera um alto acoplamento crescente com o tempo.

### Depois (respeitando o OCP)

{{< highlight python >}}
from abc import ABC, abstractmethod

class Entrega(ABC):
    @abstractmethod
    def enviar(self):
        pass


class EntregaMoto(Entrega):
    def enviar(self):
        print("Entrega realizada por moto")


class EntregaCorreios(Entrega):
    def enviar(self):
        print("Entrega enviada pelos Correios")


class EntregaDrone(Entrega):
    def enviar(self):
        print("Entrega enviada por drone")


class EntregaService:
    def processar(self, metodo: Entrega):
        print("Processando entrega...")
        metodo.enviar()
        print("Entrega concluída.")
{{< /highlight >}}


| Antes                                    | Depois                                |
| ---------------------------------------- | ------------------------------------- |
| Muitos `if/else`                         | Polimorfismo via abstração            |
| Novo tipo exige alterar código existente | Basta criar nova classe               |
| Acoplamento alto                         | Baixo acoplamento e extensibilidade   |
| Classe central conhece todos os detalhes | Classe central depende só do contrato |

**Resumo:**
A classe `EntregaService`, não conhece mais os tipos concretos. Apenas trabalha com abstração `Entrega`.  
OCP não impede mudanças, ele controla onde elas ocorrem.  
Você adiciona comportamento criando novas classes, não alterando as antigas.

## **L — Liskov Substitution Principle (Substituição de Liskov)**

> “Subtipos devem ser substituíveis por seus tipos base sem alterar o comportamento do programa.”

### Antes (violando o LSP)

{{< highlight python >}}
class Transporte:
    def carregar(self): pass
    def descarregar(self): pass
    def refrigerar(self): pass


class CaminhaoFrigorifico(Transporte):
    def carregar(self):
        print("Caminhão carregando...")
    def descarregar(self):
        print("Caminhão descarregando...")
    def refrigerar(self):
        print("Ativando refrigeração...")


class Moto(Transporte):
    def carregar(self):
        print("Moto carregando...")
    def descarregar(self):
        print("Moto descarregando...")
    def refrigerar(self):
        raise NotImplementedError("Moto não possui refrigeração!")
{{< /highlight >}}


**Problema:** A classe `Moto` herda um método que não faz sentido em ser utilizar, pois não possui refrigeração.  
Se um código espera um `Transporte` completo (com `refrigerar()`), a moto quebra o programa, violando diretamente o LSP.

### Depois (respeitando o LSP + ISP)

{{< highlight python >}}
from abc import ABC, abstractmethod

class Transporte(ABC):
    @abstractmethod
    def carregar(self): pass

    @abstractmethod
    def descarregar(self): pass


class Refrigerado(ABC):
    @abstractmethod
    def refrigerar(self): pass


class CaminhaoFrigorifico(Transporte, Refrigerado):
    def carregar(self):
        print("Caminhão frigorífico carregando...")

    def descarregar(self):
        print("Caminhão frigorífico descarregando...")

    def refrigerar(self):
        print("Refrigeração ativada")


class Moto(Transporte):
    def carregar(self):
        print("Moto carregando...")

    def descarregar(self):
        print("Moto descarregando...")


def iniciar(transporte: Transporte):
    transporte.carregar()
    transporte.descarregar()

iniciar(Moto())
iniciar(CaminhaoFrigorifico())

{{< /highlight >}}


| Antes                 | Depois                         |
| --------------------- | ------------------------------ |
| Herança forçada       | Contratos específicos e coesos |
| Exceção inesperada    | Comportamento previsível       |
| Polimorfismo quebrado | Substituição segura            |
| Superclasse inflada   | Interfaces focadas (LSP + ISP) |


**Resumo:**
LSP é sobre confiabilidade do polimorfismo.  
Se a subclasse muda as regras do tipo base, o contrato foi quebrado.

## **I — Interface Segregation Principle (Segregação de Interface)**

> “Nenhum cliente deve depender de métodos que não usa.”

### Antes (violando o ISP)

{{< highlight python >}}
class Transporte:
    def carregar(self): pass
    def descarregar(self): pass
    def refrigerar(self): pass
    def entrega_expressa(self): pass  

class Caminhao(Transporte):
    def carregar(self):
        print("Caminhão carregando...")
    def descarregar(self):
        print("Caminhão descarregando...")
    def refrigerar(self):
        print("Ativando refrigeração...")
	def entrega_expressa(self):
        raise NotImplementedError("Caminhão não faz entrega expressa!")

class Moto(Transporte):
    def carregar(self):
        print("Moto carregando...")
    def descarregar(self):
        print("Moto descarregando...")
    def refrigerar(self):
        raise NotImplementedError("Moto não possui refrigeração!")
    def entrega_expressa(self):
        raise NotImplementedError("Moto não faz entrega expressa!")

{{< /highlight >}}


**Problema:** Interface `Transporte` obriga todos os transportes a implementarem métodos que não fazem parte de sua responsabilidade.  
Surgem implementações artificiais, exceções e contratos falsos.  
Ou seja, exatamente o que o ISP evita.

### Depois (respeitando o ISP)

{{< highlight python >}}
from abc import ABC, abstractmethod

class Transporte(ABC):
    @abstractmethod
    def entregar(self): pass


class Refrigerado(ABC):
    @abstractmethod
    def refrigerar(self): pass


class Express(ABC):
    @abstractmethod
    def entrega_expressa(self): pass


class Moto(Transporte):
    def entregar(self):
        print("Moto entregando...")


class CaminhaoFrigorifico(Transporte, Refrigerado):
    def entregar(self):
        print("Caminhão frigorífico entregando...")

    def refrigerar(self):
        print("Refrigeração ativada.")


class MotoExpress(Transporte, Express):
    def entregar(self):
        print("Moto expressa entregando...")

    def entrega_expressa(self):
        print("Entrega expressa realizada!")

Moto().entregar()
MotoExpress().entrega_expressa()
CaminhaoFrigorifico().refrigerar()
{{< /highlight >}}

| Antes                            | Depois                               |
| -------------------------------- | ------------------------------------ |
| Interface genérica demais        | Interfaces pequenas e focadas        |
| Métodos sem sentido              | Contratos coesos                     |
| Classes dependem do que não usam | Cada classe implementa o que precisa |
| Exceções desnecessárias          | Comportamento válido                 |

**Resumo:**
ISP é SRP aplicado às interfaces: um motivo para mudar = um cliente que a usa.

## **D — Dependency Inversion Principle (Inversão de Dependência)**

> “Dependa de abstrações, não de implementações concretas.”

### Antes (violando o DIP)

{{< highlight python >}}
class PagamentoCartao:
    def pagar(self, valor):
        print(f"Pagando {valor} com cartão...")


class Checkout:
    def __init__(self):
        self.pagamento = PagamentoCartao()  # acoplado

    def finalizar(self, valor):
        self.pagamento.pagar(valor)


{{< /highlight >}}


**Problema:** O `Checkout` depende diretamente de um detalhe concreto: `PagamentoCartao`.  
Se surgir `Pix`, `Boleto`, `PayPal`, `criptomoeda`, `QRCode`, `transferência bancária` etc., será necessário alterar a classe `Checkout`.

### Depois (respeitando o DIP)

{{< highlight python >}}
from abc import ABC, abstractmethod

class Pagamento(ABC):
    @abstractmethod
    def pagar(self, valor): pass


class PagamentoCartao(Pagamento):
    def pagar(self, valor):
        print(f"Pagando {valor} com cartão...")


class PagamentoPix(Pagamento):
    def pagar(self, valor):
        print(f"Pagando {valor} com Pix...")


class Checkout:
    def __init__(self, metodo: Pagamento):
        self.metodo = metodo  # depende da abstração

    def finalizar(self, valor):
        self.metodo.pagar(valor)
        
checkout = Checkout(PagamentoPix())
checkout.finalizar(150)
{{< /highlight >}}


| Antes                             | Depois                       |
| --------------------------------- | ---------------------------- |
| Alto nível depende do baixo nível | Ambos dependem de abstrações |
| Implementação fixa e rígida       | Implementação plugável       |
| Testes difíceis                   | Fácil simular dependências   |
 
 **Resumo:**
O DIP define a direção da dependência.  
O alto nível não conhece detalhes de pagamento, transporte, banco, serviço externo, driver, etc.

**Resumo geral:**

| Princípio | O que muda no código             | Benefício                 |
| --------- | -------------------------------- | ------------------------- |
| **SRP**   | Uma responsabilidade por classe  | Clareza e isolamento      |
| **OCP**   | Extensão via abstração           | Código estável e flexível |
| **LSP**   | Subclasses previsíveis           | Polimorfismo confiável    |
| **ISP**   | Interfaces pequenas              | Baixo acoplamento         |
| **DIP**   | Alto nível depende de abstrações | Arquitetura limpa         |

## **Exercício final: Central de Notificações**

Crie um sistema que envie mensagens por **Email**, **SMS** e **Push**, aplicando SOLID:

1. Interface `Notificacao` define o contrato.
2. `EmailNotificacao`, `SMSNotificacao` e `PushNotificacao` implementam o contrato.
3. `CentralDeNotificacoes` injeta a abstração e envia mensagens.
4. Novos tipos podem ser adicionados sem modificar a central (OCP).
5. Cada classe tem uma responsabilidade clara (SRP).
6. A central não conhece detalhes concretos (DIP).

O gabarito do exercício está escrito em Java, disponivel no meu Github:  https://github.com/biaggiorizzo/biaggio-blog-code/tree/main/concepts/poo/solid

## Conclusão

Esse  exemplos  mostra como SOLID não é só teoria, mas é a base de arquiteturas sustentáveis, onde mudanças são simples e seguras.

# Referências

- https://medium.com/desenvolvendo-com-paixao/o-que-%C3%A9-solid-o-guia-completo-para-voc%C3%AA-entender-os-5-princ%C3%ADpios-da-poo-2b937b3fc530
- https://www.freecodecamp.org/portuguese/news/os-principios-solid-da-programacao-orientada-a-objetos-explicados-em-bom-portugues/
- https://www.youtube.com/playlist?list=PLAgbpJQADBGK8EemmTEoIwW08bXCjfSuA
- https://medium.com/backticks-tildes/the-s-o-l-i-d-principles-in-pictures-b34ce2f1e898
- https://www.youtube.com/watch?v=xNgXAQjICpQ
- https://web.archive.org/web/20191116231621/https://fi.ort.edu.uy/innovaportal/file/2032/1/design_principles.pdf