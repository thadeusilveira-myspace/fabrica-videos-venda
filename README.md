# Fábrica de vídeos curtos de venda — pacote semente

Uma live de vendas de 1 a 3 horas entra. Saem **10 a 15 vídeos verticais** de 25 a 50 segundos,
prontos para postar no TikTok Shop, com câmera que se mexe e prova de que nenhum fala preço.

Referência real: **uma live de 1h45 virou 13 vídeos em 1 hora e meia**, do arquivo cru ao link
de revisão, sem ninguém cortar nada à mão.

**Isto não é o código da fábrica.** É o que o seu Claude Code precisa ler para construir a
mesma coisa do zero, sem gastar as semanas que a primeira gastou aprendendo.

---

## Começar (2 comandos)

```bash
git clone <ENDEREÇO-DESTE-REPOSITORIO> fabrica-videos
cp -R fabrica-videos/skill/fabrica-de-videos ~/.claude/skills/
```

Depois, na pasta do seu projeto, abra o Claude Code e digite:

```
/fabrica-de-videos
```

Ele entra sabendo a ordem de leitura, os critérios de aceite de cada marco e as armadilhas
que o esperam. Você não precisa lembrar de nada disso.

**Sem a skill, funciona igual** — só exige disciplina: copie `CLAUDE.md` para a raiz do seu
projeto e mande o Claude ler `PEDRAS.md` antes de escrever a primeira linha.

---

## O que tem aqui

| arquivo | serve para |
| --- | --- |
| `LEIA-PRIMEIRO.md` | o mapa, em uma página |
| `CLAUDE.md` | **as 8 leis** — vai na raiz do seu projeto |
| `CONSTRUIR.md` | **7 marcos**, cada um com critério que passa ou não passa |
| `PEDRAS.md` | **15 armadilhas** com sintoma, causa e conserto |
| `PADRAO-DE-EDICAO.md` | a receita visual e o formato do "momento" |
| `skill/` | a mesma coisa, embrulhada como skill do Claude Code |
| `pagina.html` | tudo isso como página navegável, para ler ou apresentar |

## Por onde começar a leitura, se você só tiver 10 minutos

`PEDRAS.md`. Não é a parte mais importante do sistema, mas é a única que **você não descobre
sozinho a tempo**. Onze das quinze armadilhas são a mesma coisa vestida de roupas diferentes:
uma falha que se disfarça de outra coisa.

## O que este pacote deliberadamente não resolve

- **Não diz quais vídeos vendem.** Não há, aqui, nenhuma ligação de volta com o resultado da
  loja. Saber qual gancho converteu continua sendo trabalho manual de quem opera.
- **Não escreve bom texto de tela sozinho.** A máquina dá um palpite a partir da fala e acerta
  talvez metade. É a parte que mais precisa de olho humano.
- **Não julga se o produto pode ser anunciado.** Isso é decisão de quem opera a loja.

## Como saber se a sua fábrica ficou boa

> Entregue uma live de 1 hora a quem nunca usou o sistema. Ele deve devolver os vídeos com a
> prova de assunto proibido zerada **sem precisar perguntar nada a ninguém**.

Se ele perguntar "como eu provo que não fala preço?" ou "onde vejo se deu erro?", falta a
lei 1 ou a lei 3.
