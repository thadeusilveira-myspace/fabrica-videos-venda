# Fábrica de vídeos curtos de venda — a constituição

Este arquivo vai na raiz do projeto. São as leis. Valem qualquer que seja a linguagem,
a biblioteca ou o jeito de programar. Código se troca; isto não.

---

## LEI 1 — O que é dito no vídeo pronto se PROVA, não se confia

A fábrica escolhe trechos de fala que não tocam num assunto proibido (preço, nome de cliente,
prazo). Isso não basta.

**No fim, cada arquivo FINAL é transcrito de novo, do zero, e a fala é varrida atrás das
palavras proibidas. O resultado tem que ser zero.**

Por que isso não é exagero: entre escolher o trecho e gerar o vídeo há meia dúzia de passos
(ajuste fino de corte, emenda, aceleração) e qualquer um deles pode puxar meio segundo a mais
e trazer "cento e vinte e dois reais" para dentro do quadro. Já aconteceu: um ajustador de
corte que desloca o ponto em até 0,9 s trouxe um valor para dentro de um vídeo aprovado.

Corolário: **a prova roda no arquivo, não no plano.** Se o arquivo não existe, o resultado é
"arquivo não existe" — nunca "está limpo" e nunca "fala preço".

## LEI 2 — Nada vai ao ar sozinho

A máquina edita sozinha. A máquina **não** posta, **não** publica e **não** entrega.

O que ela produz cai numa pasta de espera (`a-revisar`) e só sai de lá por um comando explícito
que registra **quem** liberou e **quando**. Uma fila de revisão sem comando de saída é uma fila
de mentira: ninguém aprova, todo mundo arrasta arquivo no Finder, e ninguém sabe o que foi
aprovado por quê.

## LEI 3 — Silêncio é o pior estado possível

Se a máquina não entendeu alguma coisa, ela **diz**. Nunca fica quieta.

Pasta com formato que ela não lê, gravação deitada, material que não é do tipo que ela edita,
disco cheio: tudo isso vira uma mensagem escrita **onde quem mandou o arquivo vai olhar** —
dentro da própria pasta dele, não num log que só o programador abre.

Uma pasta parada sem explicação é o defeito mais caro de todos, porque ninguém descobre que
existe até alguém perguntar "cadê meus vídeos?".

## LEI 4 — Cada falha tem nome próprio

Render que falhou, transcrição que não rodou e vídeo que fala preço são **três** problemas
diferentes e precisam de **três** mensagens diferentes.

O erro clássico: a verificação não acha o arquivo (porque o render morreu) e anuncia
"este vídeo fala preço". O dono vai procurar preço numa pasta vazia e perde a confiança no
sistema inteiro.

Caso particular que vale escrever: **se a transcrição falhou, nunca concluir que a fala é
proibida.** Isso acusa a pessoa que gravou de um defeito que é da máquina.

## LEI 5 — Medir antes de decidir

Nada é presumido sobre uma gravação. Antes de cortar qualquer coisa, medir:

- **tamanho e rotação** — celular grava em pé e grava a marca `rotation: -90`; quem lê só
  largura e altura conclui "deitado" e enquadra tudo errado;
- **atraso entre imagem e som** — é diferente em cada gravação (já vimos de −0,04 s a +0,85 s)
  e é **por arquivo**, não por leva;
- **espelhamento** — câmera frontal inverte a imagem e o rótulo do produto sai ao contrário.
  Só dá para saber procurando **texto dentro da cena** (marca, embalagem, cartaz). Duas lives
  do mesmo dia podem vir uma espelhada e a outra não;
- **fala** — transcrever antes de escolher, nunca depois.

## LEI 6 — Texto de tela é a parte mais frágil do sistema

É onde tudo dá errado, e é o que o cliente vê primeiro.

- **Nunca cortar no meio da palavra.** Limite de caracteres cru escreve "PERFUME SEDUTO" na
  tela. Ou a frase cabe inteira, ou tira-se a última palavra completa.
- **Não promete preço nem prazo.** "A QUALQUER MOMENTO PODE ESGOTAR" é promessa de escassez
  e não pode ir para a tela, mesmo que a pessoa tenha falado isso.
- **Não fala da plataforma.** "O TikTok filtra sua mensagem" não vende o produto.
- **Onde o texto vai depende do enquadramento**, e isso muda por creator: em plano de corpo
  inteiro o texto vai no meio; em selfie com produto na mão ele tapa o rótulo e precisa ir no
  alto; com produto grande à frente do corpo ele vai embaixo. Conferir com um quadro extraído
  **antes** de renderizar o lote inteiro.

## LEI 7 — O que a plataforma proíbe vence tudo

Existe uma lista de produtos que não podem ser anunciados, e ela é lida **pela máquina**,
com três estados e nada além deles:

| estado | o que a máquina faz |
| --- | --- |
| **PROIBIDO** | para na hora, não transcreve, não corta, não edita |
| **Conferir** | edita, mas **não publica**, e avisa |
| **Liberado** | edita e publica |
| *(fora da lista)* | trata como **Conferir** — o lado seguro |

Isso não é teoria: um vídeo foi removido **20 minutos** depois de postado porque o produto
(máquina de solda) é proibido. Nenhuma edição conserta isso. Sinal de alerta para conferir
antes: ferramenta que corta, solda, fura, queima ou emite laser.

## LEI 8 — Falar a língua de quem usa

Mensagem na tela, nome de pasta e estado nunca são o nome da variável nem o valor cru.
Quem opera a fábrica não é quem a programou.

---

## Como saber se a fábrica está boa

Um teste, binário:

> Entregue uma live de 1 hora a quem nunca usou o sistema. Ele deve devolver os vídeos
> com prova de assunto proibido zerada **sem precisar perguntar nada a ninguém**.

Se ele perguntar "como eu provo que não fala preço?" ou "onde vejo se deu erro?",
a fábrica ainda não está pronta — falta LEI 1 ou LEI 3.
