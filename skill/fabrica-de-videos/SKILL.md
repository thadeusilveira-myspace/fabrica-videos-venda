---
name: fabrica-de-videos
description: Construir uma fábrica que transforma live de vendas longa em vídeos curtos verticais para TikTok Shop, sozinha. Use quando o pedido for montar, evoluir ou depurar esse tipo de sistema — corte automático de live, câmera virtual sobre vídeo parado, trava de assunto proibido na fala, render em lote, ou vigia de pasta que edita sem ninguém pedir.
---

# Fábrica de vídeos curtos de venda

Uma gravação de live de 1 a 3 horas entra. Saem 10 a 15 vídeos verticais de 25 a 50 segundos,
prontos para postar, com câmera que se mexe e prova de que nenhum fala preço.

Referência real: **uma live de 1h45 virou 13 vídeos em 1h30**, do arquivo cru ao link de revisão.

## Antes de escrever a primeira linha

Leia, nesta ordem, os arquivos que vieram com esta skill:

1. **`PEDRAS.md`** — 15 armadilhas já pagas, com sintoma, causa e conserto. Leia primeiro.
   Não é a parte mais importante do sistema, mas é a única que não se descobre sozinho a tempo.
   Várias não parecem bug: parecem outra coisa.
2. **`CLAUDE.md`** — as 8 leis. Copie este arquivo para a raiz do projeto.
3. **`CONSTRUIR.md`** — o plano de obra em 7 marcos.
4. **`PADRAO-DE-EDICAO.md`** — a receita visual, e o formato do "momento".

## Como conduzir a construção

- **Um marco por vez.** Cada um tem um critério que ou passa ou não passa. Prove o marco
  antes de seguir, e diga ao usuário o que foi provado e como.
- **Nunca pule o marco 0.** Ele só prova que renderizador, medidor e transcritor funcionam
  naquela máquina. Descobrir no marco 4 que falta um filtro custa dias.
- **Cada marco tem pedras esperando.** Antes de escrever o marco N, diga quais pedras se
  aplicam a ele. O `CONSTRUIR.md` aponta.
- **Prove no artefato, não no plano.** A verificação roda no arquivo de vídeo pronto, nunca
  na lista que o gerou.
- **Olhe um quadro antes de declarar pronto.** Extraia um quadro do vídeo e leia o texto da
  tela. O defeito mais feio já achado (`PERFUME SEDUTO`, palavra cortada ao meio) estava em
  10 de 13 vídeos e passou por todas as verificações automáticas, porque nenhuma olhava a tela.

## As três coisas que este sistema nunca faz

1. **Não posta e não publica sozinho.** O que ele produz espera liberação humana registrada.
2. **Não embute música.** Vídeo com link de produto só pode usar som da biblioteca comercial
   da própria plataforma; música de fora faz o vídeo ser silenciado.
3. **Não mexe em produto proibido pela plataforma.** A lista é lida pela máquina e vence tudo.

## Quando o usuário pedir para "acelerar" ou "simplificar"

Estas quatro não são negociáveis, porque cada uma existe por um estrago já acontecido:
a prova por retranscrição do arquivo final, a mensagem própria para cada tipo de falha,
o congelamento do código antes do lote, e a espera humana antes de qualquer publicação.
