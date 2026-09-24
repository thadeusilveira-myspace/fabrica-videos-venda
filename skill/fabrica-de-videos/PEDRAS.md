# As pedras

Quinze armadilhas que já custaram dias de trabalho, vídeos refeitos e uma violação de
plataforma. Cada uma tem o **sintoma** (o que você vai ver), a **causa** e o **conserto**.

Várias não parecem bug. Parecem outra coisa — e é isso que as torna caras.

---

## PEDRA 1 — o ffmpeg que vem junto com a biblioteca de render é capado

**Sintoma:** um filtro comum de ffmpeg devolve "No such filter" e você duvida da sua sintaxe.

**Causa:** o ffmpeg embutido no Remotion é uma compilação enxuta. Faltam, entre outros:
`pad`, `setsar`, `fps`, `setpts`, `volumedetect`, `hstack`, `tile`, o muxer `rawvideo` e o
encoder `null`.

**Conserto:** existem trocas para todos, e é bom conhecê-las antes de precisar —
`-r` no lugar de `fps`; `-itsscale` na entrada no lugar de `setpts` para acelerar imagem;
`loudnorm=print_format=json` no lugar de `volumedetect` para medir volume; `scale` sozinho no
lugar de `pad`+`setsar`; e mosaico de quadros se monta fora do ffmpeg.
Se preferir, instale um ffmpeg completo à parte — mas então **decida qual dos dois cada script
usa**, porque misturar os dois no mesmo projeto é uma fonte silenciosa de confusão.

---

## PEDRA 2 — a renderização morre junto com a sessão do terminal

**Sintoma:** 4 de 10 vídeos ficaram prontos e o resto sumiu sem erro. Sobra um arquivo de
trabalho pela metade que atrapalha a próxima tentativa.

**Causa:** o comando foi filho do terminal, e o terminal fechou.

**Conserto:** lote sempre destacado do terminal (`nohup` ou equivalente), com registro em
arquivo de texto e uma linha por vídeo: início, fim, deu certo ou não.

---

## PEDRA 3 — cada vídeo relê o código ao começar

**Sintoma:** você conserta uma coisa no meio de um lote e os vídeos saem misturados: os
primeiros com o código velho, os últimos com o novo. Ou pior, o lote quebra na metade.

**Causa:** o renderizador carrega o código a cada vídeo, não uma vez só.

**Conserto:** antes de começar o lote, copie o código para uma pasta congelada e renderize a
partir dela. Aí você pode mexer no código à vontade enquanto o lote roda.

---

## PEDRA 4 — o detector de "terminou de subir" que nunca dispara

**Sintoma:** o vigia repete "ainda subindo" para sempre. O arquivo está parado há uma hora.
Nenhum erro em lugar nenhum.

**Causa:** uma linha. O relógio do "está parado desde quando?" era regravado **a cada volta**
do vigia, então a conta dava sempre o intervalo da vigia (30 s) e nunca alcançava os 120 s
exigidos.

```
// errado: regrava o relógio toda volta
tamanhos.set(arquivo, { tamanho, quando: agora() });
if (antes.tamanho === tamanho && agora() - antes.quando > ESPERA) ...

// certo: o relógio só anda quando o TAMANHO muda
if (!antes || antes.tamanho !== tamanho) { tamanhos.set(arquivo, { tamanho, mudouEm: agora() }); return false; }
return agora() - antes.mudouEm >= ESPERA;
```

**Conserto, e a lição maior:** **teste detector de tempo com relógio simulado.** Rode 120
voltas com o tempo falso e veja se dispara. Esperar de verdade esconde o defeito, porque você
nunca espera uma hora olhando.

Vizinha desta pedra: usar `.every()` para conferir vários arquivos faz o laço parar no primeiro
que ainda está subindo — do segundo ao décimo terceiro nunca chegam a ser medidos.

---

## PEDRA 5 — a transcrição morre em gravação longa

**Sintoma:** o processo é morto pelo sistema, sem mensagem, em live de mais de 40 minutos.
Costuma acontecer quando a máquina está com pouca memória livre.

**Causa:** o modelo carrega o áudio inteiro.

**Conserto:** fatiar o áudio em pedaços de uns 25 minutos, transcrever um por um e juntar
somando o deslocamento de tempo de cada pedaço. Guarde cada pedaço em disco: se o processo
morrer, a retomada pula o que já foi feito.

E a regra da LEI 4: **pedaço que não transcreveu é buraco, não é fala limpa nem fala proibida.**

---

## PEDRA 6 — o zoom cego corta a cabeça

**Sintoma:** a câmera aproxima e decapita quem está falando. Ou o selo do produto tapa os
olhos dela.

**Causa:** o enquadramento foi calculado sobre o centro do quadro. Mas a pessoa anda para os
lados e chega perto da lente.

**Conserto:** detectar onde está o rosto ao longo do vídeo (no macOS, a detecção nativa do
sistema resolve e é rápida) e usar isso para duas coisas: limitar o quanto o zoom pode
aproximar, e fazer a mira acompanhar. Com duas pessoas em cena, a caixa é a **união** dos
rostos — seguir só o maior faz a câmera pular de uma para outra. E a proximidade se mede pela
**altura** do grupo: a largura dá falso positivo.

---

## PEDRA 7 — colar pedaços de áudio comprimido descola a voz da boca

**Sintoma:** no terceiro pedaço emendado, a voz está adiantada em relação à boca. Pouco, mas
o suficiente para incomodar.

**Causa:** cada arquivo de áudio comprimido carrega um "enchimento" no começo. Colando os
arquivos sem recodificar, os enchimentos se acumulam — medimos 109 ms de atraso no último
bloco de três.

**Conserto:** converter as peças para som não comprimido e colar pelo **filtro** de
concatenação, que realinha a cada emenda. Caiu para 39 ms no pior caso, sem acumular.

**E o jeito de conferir:** duração igual de imagem e som **não prova** sincronia. Mede-se por
correlação do áudio, procurando onde cada peça realmente começa — com passo fino, porque um
passo grosso faz um casamento real de 0,97 aparecer como 0,17.

---

## PEDRA 8 — o atraso entre imagem e som é por gravação

**Sintoma:** você acerta o sincronismo numa leva e na seguinte está tudo errado de novo.

**Causa:** a faixa de vídeo e a de áudio não começam no mesmo instante, e a diferença muda
por arquivo. Já vimos de −0,04 s a +0,85 s **no mesmo projeto**.

**Conserto:** medir o instante inicial das duas faixas em cada arquivo e somar essa diferença
ao ponto de corte. Guardar o valor **por arquivo**, não por leva — uma pasta com 13 gravações
tem 13 atrasos diferentes.

---

## PEDRA 9 — a imagem vem espelhada e o rótulo sai ao contrário

**Sintoma:** ninguém percebe. Depois alguém vê que a marca do produto está invertida.

**Causa:** gravação de câmera frontal.

**Conserto:** não dá para adivinhar por creator nem por plataforma — duas lives do mesmo dia
vêm uma espelhada e a outra não. O jeito é **procurar texto dentro da cena** (marca no produto,
embalagem, cartaz na parede), ampliar e ler. Quando não há texto nenhum na cena, o espelhamento
não produz erro visível: anote isso no relatório em vez de chutar.

---

## PEDRA 10 — o celular grava em pé e o arquivo diz que está deitado

**Sintoma:** o inventário classifica uma gravação vertical como deitada, e a decisão sai errada.

**Causa:** o arquivo guarda 3840×2160 **mais** uma marca de rotação de −90 graus.

**Conserto:** ler a rotação junto com largura e altura. Se houver rotação de 90 graus, o que
vale é o contrário do que as dimensões dizem.

Cuidado com a versão bug desta pedra: escrever a condição como `largura < altura === false`.
Em JavaScript isso é sempre falso — código morto que nunca reclama.

---

## PEDRA 11 — o nome que vem de fora envenena o projeto inteiro

**Sintoma:** **todos** os vídeos do lote falham em 3 segundos cada. Nenhum erro útil. O
relatório só diz "arquivo final não existe", que aponta para o lugar errado.

**Causa:** o nome da pasta do Studio veio de um nome digitado por alguém — com espaço, barra
ou acento. O Remotion só aceita letra, número e traço em nome de pasta, e rejeita o **arquivo
de composições inteiro**. Um nome ruim derruba os 150 vídeos do projeto, inclusive os que nada
têm a ver com aquela leva.

**Conserto:** limpar o nome **no código que escreve**, nunca confiar em quem digita. E guardar
o sintoma: **lote inteiro falhando em segundos = erro no arquivo de composições, não no vídeo.**

---

## PEDRA 12 — texto cortado no meio da palavra

**Sintoma:** na tela aparece "PERFUME SEDUTO". O cliente vê antes de você.

**Causa:** um limite de caracteres cru (`slice(0, 52)`).

**Conserto:** ou a frase cabe inteira, ou tira-se a última palavra completa. Melhor ainda,
nesta ordem: escolher uma frase que caiba inteira → um pedaço até a vírgula que caiba →
e só no fim, aparar pela última palavra.

**E a lição de método**, que vale mais que o conserto: **olhe um quadro do vídeo pronto antes
de dizer que está pronto.** Esse defeito estava em 10 de 13 vídeos e passou por todas as
verificações automáticas, porque nenhuma delas olhava a tela.

---

## PEDRA 13 — a fronteira de palavra não entende acento

**Sintoma:** a trava de assunto proibido reprova um vídeo limpo. A frase acusada é
"vai **real**çar a cinturinha".

**Causa:** o `\b` das expressões regulares em JavaScript só conhece `a-z`, `0-9` e `_`.
O "ç" conta como fim de palavra, então `\breal\b` casa dentro de "realçar". O mesmo
aconteceria com "pagação", "ofertaço".

**Conserto:** trocar `\b` por uma fronteira que conheça letra acentuada —
`(?<![\p{L}\p{N}_])` antes e `(?![\p{L}\p{N}_])` depois, com a bandeira `u`.

**E teste nos dois sentidos:** uma lista de frases que **têm** que ser pegas e outra de frases
que **não podem** ser. Afrouxar o filtro para matar o falso alarme é como se deixa passar preço
de verdade.

---

## PEDRA 14 — a data vem de Londres

**Sintoma:** a leva de hoje nasce carimbada com a data de amanhã.

**Causa:** `toISOString()` devolve UTC. Às 21h no Brasil já é o dia seguinte lá.

**Conserto:** montar a data com as partes locais. Parece bobagem até você ter duas levas
no lugar errado do histórico.

---

## PEDRA 15 — o conferidor que não mede aprova vídeo mudo

**Sintoma:** o relatório diz "tem som" e o vídeo está em silêncio.

**Causa:** a verificação perguntava "existe faixa de áudio?". Existe — só está vazia.

**Conserto:** **medir** o volume (LUFS) e reprovar fora da faixa. E um detalhe que custa uma
hora: a medição de volume do ffmpeg sai na **saída de erro**, não na saída padrão. Quem lê só
a saída padrão recebe vazio e conclui "não tem som".

---

## O padrão que liga quase todas

Onze das quinze pedras acima são a mesma coisa vestida de roupas diferentes:
**uma falha que se disfarça de outra coisa.**

O detector que nunca dispara parece arquivo ainda subindo. O render que morreu parece vídeo
falando preço. O acento parece filtro funcionando. O vídeo mudo parece vídeo com som.

Por isso a defesa não é escrever mais testes de unidade. É:

1. **provar no artefato final**, não no plano (LEI 1);
2. **dar nome próprio a cada falha** (LEI 4);
3. **olhar o resultado com os olhos** antes de declarar pronto — pelo menos um quadro,
   pelo menos um por lote.
