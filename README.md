<style>
#produto-sku.ga-funcoes-grade,
#produto.ga-funcoes-grade {
    width: 100%;
}

.ga-grade-acoes {
    width: 100%;
    margin: 14px 0 0;
    padding: 10px 0;
}

.ga-grade-acoes-linha {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    align-items: center;
}

.ga-grade-acoes button,
.ga-grade-acoes input,
.ga-grade-acoes select {
    height: 36px;
    border: 1px solid #d5d5d5;
    border-radius: 4px;
    background: #fff;
    font-size: 12px;
    box-sizing: border-box;
}

.ga-grade-acoes button {
    padding: 0 12px;
    cursor: pointer;
}

.ga-grade-acoes input {
    width: 65px;
    padding: 0 5px;
    text-align: center;
}

.ga-grade-acoes select {
    min-width: 78px;
    padding: 0 6px;
}

.ga-sortidos-btn {
    color: #fff !important;
    border-color: #299b16 !important;
    background: #299b16 !important;
}

.ga-grade-mensagem {
    display: block;
    min-height: 16px;
    margin-top: 5px;
    color: #777;
    font-size: 11px;
}

.ga-grade-mensagem.erro {
    color: #c00;
}

.ga-grade-resumo {
    margin-top: 8px;
    padding: 16px 12px 9px;
    background: #f1f1f1;
    text-align: center;
}

.ga-grade-resumo-itens {
    font-size: 12px;
    line-height: 16px;
}

.ga-grade-resumo-valor {
    color: #958d79;
    font-size: 24px;
    font-weight: bold;
    line-height: 28px;
}

.ga-grade-resumo-parcela {
    margin-top: 3px;
    font-size: 12px;
}

#produto-sku.ga-funcoes-grade .botoes,
#produto.ga-funcoes-grade .botoes {
    width: 100% !important;
    margin: 0 !important;
    padding: 0 !important;
}

#produto-sku.ga-funcoes-grade .row-botao-carrinho,
#produto.ga-funcoes-grade .row-botao-carrinho {
    width: 100% !important;
    margin: 0 !important;
    padding: 8px 12px 16px !important;
    background: #f1f1f1;
    box-sizing: border-box;
}

#produto-sku.ga-funcoes-grade .line-quantidades,
#produto.ga-funcoes-grade .line-quantidades {
    display: none !important;
}

#produto-sku.ga-funcoes-grade .line-botao-comprar,
#produto.ga-funcoes-grade .line-botao-comprar {
    width: 100% !important;
    max-width: 100% !important;
    flex: 0 0 100% !important;
    padding: 0 !important;
}

#produto-sku.ga-funcoes-grade .bt-comprar,
#produto.ga-funcoes-grade .bt-comprar {
    width: 100% !important;
    min-height: 42px;
}
</style>

<script>
(function () {
    "use strict";

    if (window.__funcoesGradeAtacadoWbuy) {
        return;
    }

    window.__funcoesGradeAtacadoWbuy = true;

    var PARCELAS = 6;
    var quadro;
    var observador;
    var atualizacaoAgendada = false;

    function obterRaiz() {
        var grade = document.querySelector(
            "#produto-sku .grade, #produto .grade"
        );

        if (!grade) {
            return null;
        }

        return (
            grade.closest("#produto-sku") ||
            grade.closest("#produto") ||
            grade.parentElement
        );
    }

    function obterGrade(raiz) {
        return raiz.querySelector(".grade");
    }

    function obterTamanhos(grade) {
        var itens = grade.querySelectorAll(
            ".l:first-child .vars .it"
        );

        return Array.from(itens).map(function (item, indice) {
            var texto = item.querySelector(".t p, .t");
            var nome = texto
                ? texto.textContent.trim()
                : "";

            return {
                indice: indice,
                nome: nome || "Tamanho " + (indice + 1)
            };
        });
    }

    function formatarDinheiro(valor) {
        return new Intl.NumberFormat("pt-BR", {
            style: "currency",
            currency: "BRL"
        }).format(valor);
    }

    function obterPreco(raiz) {
        var meta = raiz.querySelector(
            'meta[itemprop="price"]'
        );

        if (meta) {
            var precoMeta = Number(
                String(meta.content).replace(",", ".")
            );

            if (Number.isFinite(precoMeta)) {
                return precoMeta;
            }
        }

        var precoGlobal = Number(window.produto_valor);

        if (Number.isFinite(precoGlobal)) {
            return precoGlobal;
        }

        var texto = raiz.querySelector(
            ".valores .valor span, .valores .valor"
        );

        if (!texto) {
            return 0;
        }

        var encontrados = Array.from(
            texto.textContent.matchAll(
                /R\$\s*([\d.]+(?:,\d{1,2})?)/g
            )
        );

        if (!encontrados.length) {
            return 0;
        }

        return Number(
            encontrados[encontrados.length - 1][1]
                .replace(/\./g, "")
                .replace(",", ".")
        );
    }

    function obterQuantidade(raiz) {
        var total = 0;

        raiz.querySelectorAll(
            ".grade .vars .it input:not([type=hidden])"
        ).forEach(function (input) {
            var valor = parseInt(input.value, 10);

            if (Number.isFinite(valor) && valor > 0) {
                total += valor;
            }
        });

        return total;
    }

    function atualizarResumo(raiz) {
        if (!quadro) {
            return;
        }

        var quantidade = obterQuantidade(raiz);
        var preco = obterPreco(raiz);
        var total = quantidade * preco;
        var parcela = total / PARCELAS;

        quadro.querySelector(
            ".ga-grade-resumo-itens"
        ).textContent =
            quantidade === 1
                ? "Total de 1 item"
                : "Total de " + quantidade + " itens";

        quadro.querySelector(
            ".ga-grade-resumo-valor"
        ).textContent = formatarDinheiro(total);

        quadro.querySelector(
            ".ga-grade-resumo-parcela"
        ).textContent =
            "em até " +
            PARCELAS +
            "x de " +
            formatarDinheiro(parcela);
    }

    function dispararInput(input) {
        input.dispatchEvent(new Event("input", {
            bubbles: true
        }));
    }

    function misturar(lista) {
        for (var i = lista.length - 1; i > 0; i--) {
            var j = Math.floor(Math.random() * (i + 1));
            var temporario = lista[i];
            lista[i] = lista[j];
            lista[j] = temporario;
        }

        return lista;
    }

    function preencherTamanhos(grade) {
        var seletor = quadro.querySelector(
            ".ga-sortidos-tamanho"
        );

        var tamanhos = obterTamanhos(grade);
        var assinatura = tamanhos
            .map(function (item) {
                return item.nome;
            })
            .join("|");

        if (seletor.dataset.assinatura === assinatura) {
            return;
        }

        seletor.dataset.assinatura = assinatura;
        seletor.innerHTML = "";

        tamanhos.forEach(function (tamanho) {
            var opcao = document.createElement("option");
            opcao.value = tamanho.indice;
            opcao.textContent = tamanho.nome;
            seletor.appendChild(opcao);
        });
    }

    function criarPainel(raiz) {
        if (raiz.querySelector(".ga-grade-acoes")) {
            quadro = raiz.querySelector(".ga-grade-acoes");
            return;
        }

        quadro = document.createElement("div");
        quadro.className = "ga-grade-acoes";

        quadro.innerHTML =
            '<div class="ga-grade-acoes-linha">' +
                '<button type="button" class="ga-limpar-grade">' +
                    "Limpar grade" +
                "</button>" +
                '<input type="number" class="ga-sortidos-qtd" ' +
                    'min="1" value="1" aria-label="Quantidade">' +
                '<select class="ga-sortidos-tamanho" ' +
                    'aria-label="Tamanho"></select>' +
                '<button type="button" class="ga-sortidos-btn">' +
                    "Escolher sortidos" +
                "</button>" +
            "</div>" +
            '<span class="ga-grade-mensagem"></span>' +
            '<div class="ga-grade-resumo">' +
                '<div class="ga-grade-resumo-itens">' +
                    "Total de 0 itens" +
                "</div>" +
                '<div class="ga-grade-resumo-valor">' +
                    "R$ 0,00" +
                "</div>" +
                '<div class="ga-grade-resumo-parcela">' +
                    "em até 6x de R$ 0,00" +
                "</div>" +
            "</div>";

        var estoque = raiz.querySelector(".estoque");
        var botoes = raiz.querySelector(".botoes");

        if (estoque) {
            estoque.insertAdjacentElement(
                "afterend",
                quadro
            );
        } else if (botoes) {
            botoes.insertAdjacentElement(
                "beforebegin",
                quadro
            );
        }

        quadro.querySelector(".ga-limpar-grade")
            .addEventListener("click", function () {
                raiz.querySelectorAll(
                    ".grade .vars .it input:not([type=hidden])"
                ).forEach(function (input) {
                    if (!input.disabled) {
                        input.value = 0;
                        dispararInput(input);
                    }
                });

                quadro.querySelector(
                    ".ga-grade-mensagem"
                ).textContent = "Grade limpa.";

                atualizarResumo(raiz);
            });

        quadro.querySelector(".ga-sortidos-btn")
            .addEventListener("click", function () {
                var quantidade = parseInt(
                    quadro.querySelector(".ga-sortidos-qtd").value,
                    10
                ) || 1;

                var tamanho = parseInt(
                    quadro.querySelector(".ga-sortidos-tamanho").value,
                    10
                ) || 0;

                var celulas = [];

                var gradeAtual = obterGrade(raiz);

                gradeAtual.querySelectorAll(".l").forEach(
                    function (linha) {
                        var cor = linha.querySelector(
                            ".cor_primaria"
                        );

                        var itens = linha.querySelectorAll(
                            ".vars .it"
                        );

                        var item = itens[tamanho];

                        if (
                            !cor ||
                            !item ||
                            cor.classList.contains("sem_estoque") ||
                            item.querySelector(".aviseme")
                        ) {
                            return;
                        }

                        var input = item.querySelector(
                            "input:not([type=hidden])"
                        );

                        if (!input || input.disabled) {
                            return;
                        }

                        var maximo = parseInt(input.max, 10);

                        celulas.push({
                            input: input,
                            maximo: Number.isFinite(maximo)
                                ? maximo
                                : Infinity
                        });
                    }
                );

                celulas.forEach(function (celula) {
                    celula.input.value = 0;
                    dispararInput(celula.input);
                });

                var restante = quantidade;

                while (restante > 0 && celulas.length) {
                    misturar(celulas);

                    var avancou = false;

                    celulas.forEach(function (celula) {
                        if (restante <= 0) {
                            return;
                        }

                        var atual = parseInt(
                            celula.input.value,
                            10
                        ) || 0;

                        if (atual < celula.maximo) {
                            celula.input.value = atual + 1;
                            dispararInput(celula.input);
                            restante--;
                            avancou = true;
                        }
                    });

                    if (!avancou) {
                        break;
                    }
                }

                var mensagem = quadro.querySelector(
                    ".ga-grade-mensagem"
                );

                if (restante > 0) {
                    mensagem.className =
                        "ga-grade-mensagem erro";

                    mensagem.textContent =
                        "Só foi possível distribuir " +
                        (quantidade - restante) +
                        " peças.";
                } else {
                    mensagem.className =
                        "ga-grade-mensagem";

                    mensagem.textContent =
                        "Cores sortidas preenchidas.";
                }

                atualizarResumo(raiz);
            });
    }

    function iniciar() {
        var raiz = obterRaiz();

        if (!raiz) {
            return;
        }

        raiz.classList.add("ga-funcoes-grade");

        var grade = obterGrade(raiz);

        criarPainel(raiz);
        preencherTamanhos(grade);
        atualizarResumo(raiz);
    }

    function agendar() {
        if (atualizacaoAgendada) {
            return;
        }

        atualizacaoAgendada = true;

        window.setTimeout(function () {
            atualizacaoAgendada = false;
            iniciar();
        }, 0);
    }

    document.addEventListener("input", function (evento) {
        if (
            evento.target.closest(
                "#produto .grade input, " +
                "#produto-sku .grade input"
            )
        ) {
            agendar();
        }
    });

    document.addEventListener("click", function (evento) {
        if (
            evento.target.closest(
                "#produto .grade .plus, " +
                "#produto .grade .minus, " +
                "#produto-sku .grade .plus, " +
                "#produto-sku .grade .minus"
            )
        ) {
            window.setTimeout(agendar, 0);
        }
    });

    if (document.readyState === "loading") {
        document.addEventListener(
            "DOMContentLoaded",
            iniciar,
            { once: true }
        );
    } else {
        iniciar();
    }

    observador = new MutationObserver(agendar);

    observador.observe(document.body, {
        childList: true,
        subtree: true
    });
})();
</script>
<style>
/* Coloca os controles abaixo da grade */
#produto-sku.ga-funcoes-grade .grade.linha,
#produto.ga-funcoes-grade .grade.linha {
    float: none !important;
    display: block !important;
    width: 100% !important;
}

#produto-sku.ga-funcoes-grade .ga-grade-acoes,
#produto.ga-funcoes-grade .ga-grade-acoes {
    clear: both !important;
    display: block !important;
    width: 100% !important;

    margin: 16px 0 0 !important;
    padding: 14px 0 0 !important;

    border-top: 1px solid #dedede;
    text-align: center;
    box-sizing: border-box;
}

#produto-sku.ga-funcoes-grade .ga-grade-acoes-linha,
#produto.ga-funcoes-grade .ga-grade-acoes-linha {
    display: flex !important;
    flex-wrap: wrap;
    justify-content: center !important;
    align-items: center;
    gap: 10px !important;
}

#produto-sku.ga-funcoes-grade .ga-grade-acoes button,
#produto-sku.ga-funcoes-grade .ga-grade-acoes input,
#produto-sku.ga-funcoes-grade .ga-grade-acoes select,
#produto.ga-funcoes-grade .ga-grade-acoes button,
#produto.ga-funcoes-grade .ga-grade-acoes input,
#produto.ga-funcoes-grade .ga-grade-acoes select {
    height: 38px !important;
    border: 1px solid #d2d2d2 !important;
    border-radius: 5px !important;
    box-sizing: border-box;
}

#produto-sku.ga-funcoes-grade .ga-limpar-grade,
#produto.ga-funcoes-grade .ga-limpar-grade {
    min-width: 110px;
    padding: 0 14px !important;
    background: #fff !important;
    color: #444 !important;
}

#produto-sku.ga-funcoes-grade .ga-sortidos-qtd,
#produto.ga-funcoes-grade .ga-sortidos-qtd {
    width: 78px !important;
    padding: 0 8px !important;
    text-align: center;
}

#produto-sku.ga-funcoes-grade .ga-sortidos-tamanho,
#produto.ga-funcoes-grade .ga-sortidos-tamanho {
    min-width: 95px;
    padding: 0 8px !important;
}

#produto-sku.ga-funcoes-grade .ga-sortidos-btn,
#produto.ga-funcoes-grade .ga-sortidos-btn {
    min-width: 145px;
    padding: 0 15px !important;

    background: #299b16 !important;
    border-color: #299b16 !important;
    color: #fff !important;
}

#produto-sku.ga-funcoes-grade .ga-grade-resumo,
#produto.ga-funcoes-grade .ga-grade-resumo {
    width: 100% !important;
    margin-top: 14px !important;
    border-radius: 5px;
}

@media (max-width: 480px) {
    #produto-sku.ga-funcoes-grade .ga-grade-acoes-linha,
    #produto.ga-funcoes-grade .ga-grade-acoes-linha {
        gap: 7px !important;
    }

    #produto-sku.ga-funcoes-grade .ga-grade-acoes-linha > *,
    #produto.ga-funcoes-grade .ga-grade-acoes-linha > * {
        flex: 1 1 calc(50% - 7px);
    }

    #produto-sku.ga-funcoes-grade .ga-sortidos-btn,
    #produto.ga-funcoes-grade .ga-sortidos-btn {
        flex-basis: 100%;
    }
}
</style>
<style>
#produto-sku.ga-funcoes-grade .ga-grade-resumo-parcela,
#produto.ga-funcoes-grade .ga-grade-resumo-parcela {
    display: none !important;
}
</style>
<script>
(function () {
    function preencherSortido() {
        var select = document.querySelector(
            "#produto .ga-sortidos-tamanho, " +
            "#produto-sku .ga-sortidos-tamanho"
        );

        if (!select) return;

        var tamanhos = [];

        document.querySelectorAll(
            ".ga-tamanhos-cabecalho span, " +
            "#produto .grade .l:first-child .vars .it .t p, " +
            "#produto-sku .grade .l:first-child .vars .it .t p"
        ).forEach(function (elemento) {
            var texto = elemento.textContent.trim();

            if (
                texto &&
                texto.length <= 8 &&
                !tamanhos.includes(texto)
            ) {
                tamanhos.push(texto);
            }
        });

        if (!tamanhos.length) return;

        var valorAnterior = select.value;
        select.innerHTML =
            '<option value="">Tamanho</option>';

        tamanhos.forEach(function (tamanho, indice) {
            var option = document.createElement("option");
            option.value = indice;
            option.textContent = tamanho;
            select.appendChild(option);
        });

        if (valorAnterior !== "") {
            select.value = valorAnterior;
        }
    }

    preencherSortido();

    var tentativas = 0;
    var intervalo = setInterval(function () {
        preencherSortido();
        tentativas++;

        if (tentativas >= 15) {
            clearInterval(intervalo);
        }
    }, 500);
})();
</script>
<style>
.ga-percentuais-grade {
    display: flex;
    flex-wrap: wrap;
    gap: 7px;
    margin-top: 8px;
}

.ga-percentuais-grade button {
    flex: 1 1 22%;
    min-height: 34px;
    border: 1px solid #299b16;
    border-radius: 4px;
    background: #fff;
    color: #299b16;
    cursor: pointer;
    font-size: 12px;
    font-weight: 600;
}

.ga-percentuais-grade button:hover,
.ga-percentuais-grade button.ativo {
    background: #299b16;
    color: #fff;
}
</style>

<script>
(function () {
    if (window.__percentuaisGradeWbuy) return;
    window.__percentuaisGradeWbuy = true;

    var percentuais = [
        { nome: "P", valor: 10 },
        { nome: "M", valor: 35 },
        { nome: "G", valor: 35 },
        { nome: "GG", valor: 20 }
    ];

    function embaralhar(lista) {
        return lista.sort(function () {
            return Math.random() - 0.5;
        });
    }

    function localizarIndiceTamanho(grade, tamanho) {
        var encontrados = [];

        grade.querySelectorAll(
            ".l:first-child .vars .it .t p, " +
            ".ga-tamanhos-cabecalho span"
        ).forEach(function (item) {
            encontrados.push(item.textContent.trim().toUpperCase());
        });

        return encontrados.indexOf(tamanho);
    }

    function distribuir(grade, indice, quantidade) {
        var celulas = [];

        grade.querySelectorAll(".l").forEach(function (linha) {
            var itens = linha.querySelectorAll(".vars .it");
            var item = itens[indice];

            if (!item) return;

            var input = item.querySelector(
                'input:not([type="hidden"])'
            );

            var cor = linha.querySelector(".cor_primaria");

            if (
                !input ||
                input.disabled ||
                !cor ||
                cor.classList.contains("sem_estoque") ||
                item.querySelector(".aviseme")
            ) {
                return;
            }

            var maximo = parseInt(input.max, 10);

            celulas.push({
                input: input,
                maximo: Number.isFinite(maximo) ? maximo : Infinity
            });
        });

        celulas.forEach(function (celula) {
            celula.input.value = 0;
            celula.input.dispatchEvent(
                new Event("input", { bubbles: true })
            );
        });

        var restante = quantidade;

        while (restante > 0 && celulas.length) {
            embaralhar(celulas);

            var alterou = false;

            celulas.forEach(function (celula) {
                if (restante <= 0) return;

                var atual = parseInt(celula.input.value, 10) || 0;

                if (atual < celula.maximo) {
                    celula.input.value = atual + 1;
                    celula.input.dispatchEvent(
                        new Event("input", { bubbles: true })
                    );
                    restante--;
                    alterou = true;
                }
            });

            if (!alterou) break;
        }
    }

    function iniciar() {
        var painel = document.querySelector(
            ".ga-grade-acoes"
        );

        var grade = document.querySelector(
            "#produto .grade, #produto-sku .grade"
        );

        if (!painel || !grade) return;
        if (painel.querySelector(".ga-percentuais-grade")) return;

        var linha = document.createElement("div");
        linha.className = "ga-percentuais-grade";

        percentuais.forEach(function (item) {
            var botao = document.createElement("button");
            botao.type = "button";
            botao.textContent =
                item.nome + " " + item.valor + "%";

            botao.addEventListener("click", function () {
                var campo = painel.querySelector(
                    ".ga-sortidos-qtd"
                );

                var total = parseInt(campo.value, 10) || 0;
                var indice = localizarIndiceTamanho(
                    grade,
                    item.nome
                );

                if (total <= 0 || indice < 0) return;

                var quantidade = Math.round(
                    total * item.valor / 100
                );

                distribuir(
                    grade,
                    indice,
                    quantidade
                );

                botao.classList.add("ativo");

                setTimeout(function () {
                    botao.classList.remove("ativo");
                }, 700);
            });

            linha.appendChild(botao);
        });

        painel.appendChild(linha);
    }

    iniciar();
    setTimeout(iniciar, 500);
    setTimeout(iniciar, 1500);
})();
</script>
<style>
.ga-total-sortido-label {
    width: 100%;
    margin: 8px 0 3px;
    color: #333;
    font-size: 12px;
    font-weight: 600;
}

.ga-sortidos-qtd {
    width: 110px !important;
    text-align: center;
}
</style>

<script>
(function () {
    function configurarCampoTotal() {
        var painel = document.querySelector(".ga-grade-acoes");
        if (!painel) return;

        var campo = painel.querySelector(".ga-sortidos-qtd");
        if (!campo) return;

        campo.type = "number";
        campo.min = "1";
        campo.step = "1";
        campo.value = campo.value || "1";
        campo.placeholder = "Total de peças";
        campo.setAttribute(
            "aria-label",
            "Quantidade total de peças"
        );

        if (!painel.querySelector(".ga-total-sortido-label")) {
            var label = document.createElement("label");
            label.className = "ga-total-sortido-label";
            label.textContent =
                "Quantidade total de peças para distribuir:";
            campo.parentNode.insertBefore(label, campo);
        }
    }

    configurarCampoTotal();
    setTimeout(configurarCampoTotal, 500);
    setTimeout(configurarCampoTotal, 1500);
})();
</script>
<script>
(function () {
    "use strict";

    if (window.__gradeTamanhosUniversais) return;
    window.__gradeTamanhosUniversais = true;

    var percentuais = [10, 35, 35, 20];
    var atualizacaoPendente = false;

    function obterGrade() {
        return document.querySelector(
            "#produto-sku .grade, #produto .grade"
        );
    }

    function obterPainel() {
        return document.querySelector(".ga-grade-acoes");
    }

    function obterTamanhos(grade) {
        var tamanhos = [];

        var cabecalho = document.querySelectorAll(
            ".ga-tamanhos-cabecalho span"
        );

        if (cabecalho.length) {
            cabecalho.forEach(function (item) {
                var texto = item.textContent.trim();
                if (texto) tamanhos.push(texto);
            });

            return tamanhos;
        }

        var primeiraLinha = grade.querySelector(".l");

        if (!primeiraLinha) return tamanhos;

        primeiraLinha.querySelectorAll(
            ".vars .it"
        ).forEach(function (item, indice) {
            var elemento = item.querySelector(
                ".t p:first-child, .t, " +
                "[data-tamanho], [data-size]"
            );

            var texto = "";

            if (elemento) {
                texto =
                    elemento.getAttribute("data-tamanho") ||
                    elemento.getAttribute("data-size") ||
                    elemento.textContent.trim();
            }

            tamanhos.push(
                texto || "Tamanho " + (indice + 1)
            );
        });

        return tamanhos;
    }

    function disparar(input) {
        input.dispatchEvent(
            new Event("input", { bubbles: true })
        );

        input.dispatchEvent(
            new Event("change", { bubbles: true })
        );
    }

    function embaralhar(lista) {
        for (var i = lista.length - 1; i > 0; i--) {
            var j = Math.floor(Math.random() * (i + 1));
            var temporario = lista[i];

            lista[i] = lista[j];
            lista[j] = temporario;
        }

        return lista;
    }

    function obterCelulas(grade, indiceTamanho) {
        var celulas = [];

        grade.querySelectorAll(".l").forEach(function (linha) {
            var itens = linha.querySelectorAll(".vars .it");
            var item = itens[indiceTamanho];

            if (!item || item.querySelector(".aviseme")) {
                return;
            }

            var cor = linha.querySelector(".cor_primaria");

            if (
                cor &&
                cor.classList.contains("sem_estoque")
            ) {
                return;
            }

            var input = item.querySelector(
                'input:not([type="hidden"])'
            );

            if (!input || input.disabled) return;

            var maximo = parseInt(
                input.getAttribute("max") ||
                input.getAttribute("data-max") ||
                input.getAttribute("data-estoque"),
                10
            );

            celulas.push({
                input: input,
                maximo:
                    Number.isFinite(maximo) && maximo >= 0
                        ? maximo
                        : Infinity
            });
        });

        return celulas;
    }

    function limparTamanho(celulas) {
        celulas.forEach(function (celula) {
            celula.input.value = 0;
            disparar(celula.input);
        });
    }

    function distribuirCores(
        grade,
        indiceTamanho,
        quantidade
    ) {
        var celulas = obterCelulas(
            grade,
            indiceTamanho
        );

        limparTamanho(celulas);

        var restante = quantidade;

        while (restante > 0 && celulas.length) {
            embaralhar(celulas);

            var adicionou = false;

            celulas.forEach(function (celula) {
                if (restante <= 0) return;

                var atual =
                    parseInt(celula.input.value, 10) || 0;

                if (atual < celula.maximo) {
                    celula.input.value = atual + 1;
                    disparar(celula.input);

                    restante--;
                    adicionou = true;
                }
            });

            if (!adicionou) break;
        }

        return quantidade - restante;
    }

    function calcularQuantidades(total) {
        var resultado = [];
        var fracoes = [];
        var utilizado = 0;

        percentuais.forEach(function (percentual, indice) {
            var quantidadeExata =
                total * percentual / 100;

            var quantidadeInteira =
                Math.floor(quantidadeExata);

            resultado.push(quantidadeInteira);

            fracoes.push({
                indice: indice,
                fracao:
                    quantidadeExata - quantidadeInteira
            });

            utilizado += quantidadeInteira;
        });

        fracoes.sort(function (a, b) {
            return b.fracao - a.fracao;
        });

        var restante = total - utilizado;
        var posicao = 0;

        while (restante > 0) {
            resultado[
                fracoes[posicao % fracoes.length].indice
            ]++;

            restante--;
            posicao++;
        }

        return resultado;
    }

    function mostrarMensagem(texto, erro) {
        var painel = obterPainel();

        if (!painel) return;

        var mensagem =
            painel.querySelector(".ga-grade-mensagem") ||
            painel.querySelector(".ga-sortidos-mensagem");

        if (!mensagem) return;

        mensagem.textContent = texto;
        mensagem.classList.toggle("erro", Boolean(erro));
    }

    function obterQuantidadeInformada(painel) {
        var campo = painel.querySelector(
            ".ga-sortidos-qtd"
        );

        return campo
            ? parseInt(campo.value, 10) || 0
            : 0;
    }

    function configurarSeletor(grade, painel) {
        var select = painel.querySelector(
            ".ga-sortidos-tamanho"
        );

        if (!select) return;

        var tamanhos = obterTamanhos(grade);
        var assinatura = tamanhos.join("|");

        if (select.dataset.tamanhos === assinatura) {
            return;
        }

        select.dataset.tamanhos = assinatura;
        select.innerHTML =
            '<option value="">Tamanho</option>';

        tamanhos.forEach(function (tamanho, indice) {
            var opcao = document.createElement("option");

            opcao.value = indice;
            opcao.textContent = tamanho;

            select.appendChild(opcao);
        });
    }

    function configurarSortido(grade, painel) {
        var botaoAntigo = painel.querySelector(
            ".ga-sortidos-btn"
        );

        if (
            !botaoAntigo ||
            botaoAntigo.dataset.corrigido === "sim"
        ) {
            return;
        }

        /* Remove os eventos antigos desse botão */
        var botao = botaoAntigo.cloneNode(true);

        botao.dataset.corrigido = "sim";
        botaoAntigo.replaceWith(botao);

        botao.addEventListener("click", function () {
            var select = painel.querySelector(
                ".ga-sortidos-tamanho"
            );

            var quantidade =
                obterQuantidadeInformada(painel);

            var indice =
                select && select.value !== ""
                    ? parseInt(select.value, 10)
                    : -1;

            if (quantidade <= 0) {
                mostrarMensagem(
                    "Informe a quantidade de peças.",
                    true
                );
                return;
            }

            if (indice < 0) {
                mostrarMensagem(
                    "Escolha um tamanho.",
                    true
                );
                return;
            }

            var preenchidas = distribuirCores(
                grade,
                indice,
                quantidade
            );

            if (preenchidas < quantidade) {
                mostrarMensagem(
                    "Foram distribuídas " +
                    preenchidas +
                    " de " +
                    quantidade +
                    " peças por falta de estoque.",
                    true
                );
            } else {
                mostrarMensagem(
                    "Cores sortidas preenchidas.",
                    false
                );
            }
        });
    }

    function configurarPercentuais(grade, painel) {
        var area = painel.querySelector(
            ".ga-percentuais-grade"
        );

        if (!area) return;

        var tamanhos = obterTamanhos(grade).slice(0, 4);
        var assinatura = tamanhos.join("|");

        if (
            area.dataset.corrigida === assinatura &&
            area.querySelectorAll("button").length ===
                tamanhos.length
        ) {
            return;
        }

        area.dataset.corrigida = assinatura;
        area.innerHTML = "";

        tamanhos.forEach(function (tamanho, indice) {
            var botao = document.createElement("button");

            botao.type = "button";
            botao.textContent =
                tamanho + " " + percentuais[indice] + "%";

            botao.addEventListener("click", function () {
                var total =
                    obterQuantidadeInformada(painel);

                if (total <= 0) {
                    mostrarMensagem(
                        "Informe a quantidade total de peças.",
                        true
                    );
                    return;
                }

                var quantidades =
                    calcularQuantidades(total);

                var desejada = quantidades[indice];

                var preenchidas = distribuirCores(
                    grade,
                    indice,
                    desejada
                );

                if (preenchidas < desejada) {
                    mostrarMensagem(
                        tamanho +
                        ": foram distribuídas " +
                        preenchidas +
                        " de " +
                        desejada +
                        " peças por falta de estoque.",
                        true
                    );
                } else {
                    mostrarMensagem(
                        tamanho +
                        ": " +
                        preenchidas +
                        " peças distribuídas.",
                        false
                    );
                }
            });

            area.appendChild(botao);
        });
    }

    function iniciar() {
        var grade = obterGrade();
        var painel = obterPainel();

        if (!grade || !painel) return;

        configurarSeletor(grade, painel);
        configurarSortido(grade, painel);
        configurarPercentuais(grade, painel);
    }

    function agendar() {
        if (atualizacaoPendente) return;

        atualizacaoPendente = true;

        setTimeout(function () {
            atualizacaoPendente = false;
            iniciar();
        }, 100);
    }

    iniciar();
    setTimeout(iniciar, 500);
    setTimeout(iniciar, 1500);

    new MutationObserver(agendar).observe(
        document.body,
        {
            childList: true,
            subtree: true
        }
    );
})();
</script>
<style>
/* Esconde os botões percentuais antigos */
.ga-percentuais-grade {
    display: none !important;
}

/* Nova distribuição percentual */
.ga-percentuais-editaveis {
    width: 100%;
    margin: 14px 0;
}

.ga-percentuais-titulo {
    margin-bottom: 10px;
    color: #333;
    font-size: 12px;
    font-weight: 600;
    text-align: center;
}

.ga-percentuais-campos {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 8px;
}

.ga-percentual-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    min-width: 0;
}

.ga-percentual-tamanho {
    display: block;
    margin-bottom: 5px;
    color: #222;
    font-size: 13px;
    font-weight: 700;
    text-align: center;
}

.ga-percentual-caixa {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 72px;
    height: 40px;
    border: 1px solid #d6d6d6;
    border-radius: 5px;
    background: #fff;
}

.ga-percentual-input {
    width: 43px !important;
    height: 36px !important;
    padding: 0 !important;
    border: 0 !important;
    background: transparent !important;
    color: #222 !important;
    font-size: 13px !important;
    font-weight: 600;
    text-align: right !important;
    outline: none;
}

.ga-percentual-simbolo {
    margin-left: 2px;
    color: #299b16;
    font-size: 13px;
    font-weight: 700;
}

.ga-percentual-status {
    display: block;
    min-height: 15px;
    margin-top: 4px;
    color: #777;
    font-size: 10px;
    line-height: 12px;
    text-align: center;
}

.ga-percentual-status.falta {
    color: #d00000;
    font-weight: 600;
}

.ga-percentuais-total {
    margin: 8px 0;
    color: #555;
    font-size: 11px;
    text-align: center;
}

.ga-percentuais-total.erro {
    color: #d00000;
    font-weight: 600;
}

.ga-aplicar-percentuais {
    display: block;
    width: 100%;
    min-height: 38px;
    border: 0;
    border-radius: 4px;
    background: #299b16;
    color: #fff;
    cursor: pointer;
    font-size: 13px;
    font-weight: 600;
}

.ga-aplicar-percentuais:disabled {
    background: #aaa;
    cursor: not-allowed;
}

@media (max-width: 480px) {
    .ga-percentuais-campos {
        gap: 5px;
    }

    .ga-percentual-caixa {
        width: 62px;
    }
}
</style>

<script>
(function () {
    "use strict";

    if (window.__percentuaisEditaveisWbuy) return;
    window.__percentuaisEditaveisWbuy = true;

    var PADRAO = [10, 35, 35, 20];
    var atualizacaoAgendada = false;

    function obterGrade() {
        return document.querySelector(
            "#produto-sku .grade, #produto .grade"
        );
    }

    function obterPainel() {
        return document.querySelector(".ga-grade-acoes");
    }

    function obterTamanhos(grade) {
        var tamanhos = [];

        var cabecalho = document.querySelectorAll(
            ".ga-tamanhos-cabecalho span"
        );

        if (cabecalho.length) {
            cabecalho.forEach(function (elemento) {
                var texto = elemento.textContent.trim();

                if (texto) tamanhos.push(texto);
            });
        }

        if (tamanhos.length) {
            return tamanhos.slice(0, 4);
        }

        var primeiraLinha = grade.querySelector(".l");

        if (!primeiraLinha) return [];

        primeiraLinha.querySelectorAll(
            ".vars .it"
        ).forEach(function (item, indice) {
            var elemento = item.querySelector(
                ".t p:first-child, .t, " +
                "[data-tamanho], [data-size]"
            );

            var texto = "";

            if (elemento) {
                texto =
                    elemento.getAttribute("data-tamanho") ||
                    elemento.getAttribute("data-size") ||
                    elemento.textContent.trim();
            }

            tamanhos.push(
                texto || "Tamanho " + (indice + 1)
            );
        });

        return tamanhos.slice(0, 4);
    }

    function disparar(input) {
        input.dispatchEvent(
            new Event("input", { bubbles: true })
        );

        input.dispatchEvent(
            new Event("change", { bubbles: true })
        );
    }

    function embaralhar(lista) {
        for (var i = lista.length - 1; i > 0; i--) {
            var j = Math.floor(Math.random() * (i + 1));
            var auxiliar = lista[i];

            lista[i] = lista[j];
            lista[j] = auxiliar;
        }

        return lista;
    }

    function obterCelulas(grade, indice) {
        var celulas = [];

        grade.querySelectorAll(".l").forEach(function (linha) {
            var item = linha.querySelectorAll(
                ".vars .it"
            )[indice];

            if (!item || item.querySelector(".aviseme")) {
                return;
            }

            var cor = linha.querySelector(".cor_primaria");

            if (
                cor &&
                cor.classList.contains("sem_estoque")
            ) {
                return;
            }

            var input = item.querySelector(
                'input:not([type="hidden"])'
            );

            if (!input || input.disabled) return;

            var maximo = parseInt(
                input.getAttribute("max") ||
                input.getAttribute("data-max") ||
                input.getAttribute("data-estoque"),
                10
            );

            celulas.push({
                input: input,
                maximo:
                    Number.isFinite(maximo) && maximo >= 0
                        ? maximo
                        : Infinity
            });
        });

        return celulas;
    }

    function limparCelulas(celulas) {
        celulas.forEach(function (celula) {
            celula.input.value = 0;
            disparar(celula.input);
        });
    }

    function preencherAleatoriamente(
        celulas,
        quantidade
    ) {
        var restante = quantidade;

        while (restante > 0 && celulas.length) {
            embaralhar(celulas);

            var adicionou = false;

            celulas.forEach(function (celula) {
                if (restante <= 0) return;

                var atual =
                    parseInt(celula.input.value, 10) || 0;

                if (atual < celula.maximo) {
                    celula.input.value = atual + 1;
                    disparar(celula.input);

                    restante--;
                    adicionou = true;
                }
            });

            if (!adicionou) break;
        }

        return quantidade - restante;
    }

    function calcularDivisao(total, percentuais) {
        var resultado = [];
        var fracoes = [];
        var distribuido = 0;

        percentuais.forEach(function (percentual, indice) {
            var exato = total * percentual / 100;
            var inteiro = Math.floor(exato);

            resultado.push(inteiro);
            distribuido += inteiro;

            fracoes.push({
                indice: indice,
                valor: exato - inteiro
            });
        });

        fracoes.sort(function (a, b) {
            return b.valor - a.valor;
        });

        var restante = total - distribuido;
        var posicao = 0;

        while (restante > 0) {
            resultado[
                fracoes[posicao % fracoes.length].indice
            ]++;

            restante--;
            posicao++;
        }

        return resultado;
    }

    function atualizarSoma(area) {
        var inputs = area.querySelectorAll(
            ".ga-percentual-input"
        );

        var soma = 0;

        inputs.forEach(function (input) {
            soma += parseInt(input.value, 10) || 0;
        });

        var resumo = area.querySelector(
            ".ga-percentuais-total"
        );

        var botao = area.querySelector(
            ".ga-aplicar-percentuais"
        );

        if (soma === 100) {
            resumo.textContent = "Total: 100%";
            resumo.classList.remove("erro");
            botao.disabled = false;
        } else if (soma < 100) {
            resumo.textContent =
                "Faltam " + (100 - soma) + "%";

            resumo.classList.add("erro");
            botao.disabled = true;
        } else {
            resumo.textContent =
                "Ultrapassou " + (soma - 100) + "%";

            resumo.classList.add("erro");
            botao.disabled = true;
        }
    }

    function limitarPercentual(input, area) {
        var valor = parseInt(input.value, 10) || 0;

        valor = Math.max(0, Math.min(100, valor));

        var outros = 0;

        area.querySelectorAll(
            ".ga-percentual-input"
        ).forEach(function (outro) {
            if (outro !== input) {
                outros += parseInt(outro.value, 10) || 0;
            }
        });

        var maximoPermitido = Math.max(0, 100 - outros);

        if (valor > maximoPermitido) {
            valor = maximoPermitido;
        }

        input.value = valor;
        atualizarSoma(area);
    }

    function criarArea(grade, painel) {
        var tamanhos = obterTamanhos(grade);

        if (tamanhos.length < 4) return;

        var assinatura = tamanhos.join("|");
        var area = painel.querySelector(
            ".ga-percentuais-editaveis"
        );

        if (
            area &&
            area.dataset.tamanhos === assinatura
        ) {
            return area;
        }

        if (area) area.remove();

        area = document.createElement("div");
        area.className = "ga-percentuais-editaveis";
        area.dataset.tamanhos = assinatura;

        var titulo = document.createElement("div");
        titulo.className = "ga-percentuais-titulo";
        titulo.textContent =
            "Distribuição por tamanho e cores sortidas";

        var campos = document.createElement("div");
        campos.className = "ga-percentuais-campos";

        tamanhos.forEach(function (tamanho, indice) {
            var card = document.createElement("label");
            card.className = "ga-percentual-card";

            card.innerHTML =
                '<span class="ga-percentual-tamanho">' +
                    tamanho +
                "</span>" +
                '<span class="ga-percentual-caixa">' +
                    '<input type="number" ' +
                    'class="ga-percentual-input" ' +
                    'min="0" max="100" step="1" ' +
                    'value="' + PADRAO[indice] + '">' +
                    '<span class="ga-percentual-simbolo">%</span>' +
                "</span>" +
                '<small class="ga-percentual-status">' +
                    "?" +
                "</small>";

            campos.appendChild(card);
        });

        var soma = document.createElement("div");
        soma.className = "ga-percentuais-total";
        soma.textContent = "Total: 100%";

        var botao = document.createElement("button");
        botao.type = "button";
        botao.className = "ga-aplicar-percentuais";
        botao.textContent = "Distribuir por porcentagem";

        area.appendChild(titulo);
        area.appendChild(campos);
        area.appendChild(soma);
        area.appendChild(botao);

        var linhaPrincipal = painel.querySelector(
            ".ga-grade-acoes-linha"
        );

        if (linhaPrincipal) {
            linhaPrincipal.insertAdjacentElement(
                "afterend",
                area
            );
        } else {
            painel.insertBefore(area, painel.firstChild);
        }

        area.querySelectorAll(
            ".ga-percentual-input"
        ).forEach(function (input) {
            input.addEventListener("input", function () {
                limitarPercentual(input, area);
            });
        });

        botao.addEventListener("click", function () {
            var campoTotal = painel.querySelector(
                ".ga-sortidos-qtd"
            );

            var total = campoTotal
                ? parseInt(campoTotal.value, 10) || 0
                : 0;

            if (total <= 0) {
                soma.textContent =
                    "Informe a quantidade total de peças.";

                soma.classList.add("erro");
                return;
            }

            var percentuaisAtuais = [];

            area.querySelectorAll(
                ".ga-percentual-input"
            ).forEach(function (input) {
                percentuaisAtuais.push(
                    parseInt(input.value, 10) || 0
                );
            });

            var totalPercentual =
                percentuaisAtuais.reduce(
                    function (acumulado, valor) {
                        return acumulado + valor;
                    },
                    0
                );

            if (totalPercentual !== 100) {
                atualizarSoma(area);
                return;
            }

            var quantidades = calcularDivisao(
                total,
                percentuaisAtuais
            );

            var grupos = [];

            tamanhos.forEach(function (tamanho, indice) {
                var celulas = obterCelulas(grade, indice);

                limparCelulas(celulas);

                grupos.push({
                    tamanho: tamanho,
                    celulas: celulas,
                    desejada: quantidades[indice]
                });
            });

            var faltouAlguma = false;

            grupos.forEach(function (grupo, indice) {
                var preenchida = preencherAleatoriamente(
                    grupo.celulas,
                    grupo.desejada
                );

                var faltam =
                    grupo.desejada - preenchida;

                var status = area.querySelectorAll(
                    ".ga-percentual-status"
                )[indice];

                if (faltam > 0) {
                    status.textContent =
                        "Faltam " + faltam + " peças";

                    status.classList.add("falta");
                    faltouAlguma = true;
                } else {
                    status.textContent =
                        preenchida + " peças";

                    status.classList.remove("falta");
                }
            });

            soma.textContent = faltouAlguma
                ? "Distribuído conforme o estoque disponível"
                : "Distribuição concluída";

            soma.classList.toggle(
                "erro",
                faltouAlguma
            );
        });

        atualizarSoma(area);

        return area;
    }

    function iniciar() {
        var grade = obterGrade();
        var painel = obterPainel();

        if (!grade || !painel) return;

        criarArea(grade, painel);
    }

    function agendar() {
        if (atualizacaoAgendada) return;

        atualizacaoAgendada = true;

        setTimeout(function () {
            atualizacaoAgendada = false;
            iniciar();
        }, 100);
    }

    iniciar();
    setTimeout(iniciar, 500);
    setTimeout(iniciar, 1500);

    new MutationObserver(agendar).observe(
        document.body,
        {
            childList: true,
            subtree: true
        }
    );
})();
</script>
<style>
.ga-limpar-percentuais {
    display: block !important;
    width: 100% !important;
    min-height: 38px !important;
    margin-top: 8px !important;
    padding: 8px 14px !important;

    border: 1px solid #d4d4d4 !important;
    border-radius: 5px !important;

    background: #ffffff !important;
    color: #444444 !important;

    font-size: 12px !important;
    font-weight: 500 !important;
    text-align: center !important;

    cursor: pointer !important;
}

.ga-limpar-percentuais:hover {
    border-color: #999999 !important;
    background: #f5f5f5 !important;
    color: #222222 !important;
}
</style>

<script>
(function () {
    "use strict";

    var agendado = false;

    function instalarBotao() {
        var area = document.querySelector(
            ".ga-percentuais-editaveis"
        );

        if (
            !area ||
            area.querySelector(".ga-limpar-percentuais")
        ) {
            return;
        }

        var botaoDistribuir = area.querySelector(
            ".ga-aplicar-percentuais"
        );

        if (!botaoDistribuir) return;

        var botaoLimpar = document.createElement("button");

        botaoLimpar.type = "button";
        botaoLimpar.className =
            "ga-limpar-percentuais";

        botaoLimpar.textContent =
            "Limpar porcentagens";

        botaoDistribuir.insertAdjacentElement(
            "afterend",
            botaoLimpar
        );

        botaoLimpar.addEventListener(
            "click",
            function () {
                area.querySelectorAll(
                    ".ga-percentual-input"
                ).forEach(function (input) {
                    input.value = 0;

                    input.dispatchEvent(
                        new Event("input", {
                            bubbles: true
                        })
                    );

                    input.dispatchEvent(
                        new Event("change", {
                            bubbles: true
                        })
                    );
                });

                area.querySelectorAll(
                    ".ga-percentual-status"
                ).forEach(function (status) {
                    status.textContent = "?";
                    status.classList.remove("falta");
                });

                var total = area.querySelector(
                    ".ga-percentuais-total"
                );

                if (total) {
                    total.textContent = "Faltam 100%";
                    total.classList.add("erro");
                }

                botaoDistribuir.disabled = true;
            }
        );
    }

    function agendarInstalacao() {
        if (agendado) return;

        agendado = true;

        setTimeout(function () {
            agendado = false;
            instalarBotao();
        }, 100);
    }

    instalarBotao();
    setTimeout(instalarBotao, 500);
    setTimeout(instalarBotao, 1500);

    new MutationObserver(agendarInstalacao).observe(
        document.body,
        {
            childList: true,
            subtree: true
        }
    );
})();
</script>
<style>
.ga-modo-percentuais {
    display: flex;
    justify-content: center;
    gap: 6px;
    margin: 0 0 14px;
}
.ga-modo-percentuais button {
    min-height: 36px !important;
    padding: 7px 18px !important;
    border: 1px solid #d4d4d4 !important;
    border-radius: 5px !important;
    background: #fff !important;
    color: #444 !important;
    cursor: pointer;
}
.ga-modo-percentuais button.ativo {
    border-color: #299b16 !important;
    background: #299b16 !important;
    color: #fff !important;
}
.ga-modo-cores-percentual .ga-percentuais-campos {
    display: none !important;
}
.ga-cor-percentual-campo { display: none !important; }
#produto .grade.ga-percentual-por-cor .l .cor,
#produto-sku .grade.ga-percentual-por-cor .l .cor {
    flex: 0 0 112px !important;
    width: 112px !important;
    height: 56px !important;
    position: relative !important;
}
#produto .grade.ga-percentual-por-cor .ga-cor-percentual-campo,
#produto-sku .grade.ga-percentual-por-cor .ga-cor-percentual-campo {
    position: absolute;
    top: 0;
    right: 0;
    display: flex !important;
    flex-direction: column;
    align-items: center;
    width: 56px;
}
.ga-cor-percentual-caixa {
    display: flex;
    align-items: center;
    width: 56px;
    height: 36px;
    border: 1px solid #d4d4d4;
    border-radius: 5px;
    background: #fff;
    box-sizing: border-box;
}
#produto .grade .ga-cor-percentual,
#produto-sku .grade .ga-cor-percentual {
    width: 38px !important;
    height: 32px !important;
    min-height: 0 !important;
    margin: 0 !important;
    padding: 0 !important;
    border: 0 !important;
    float: none !important;
    background: transparent !important;
    color: #222 !important;
    font-size: 12px !important;
    text-align: center !important;
}
.ga-cor-percentual-caixa > span {
    color: #299b16;
    font-size: 11px;
}
.ga-cor-percentual-status {
    margin-top: 3px;
    color: #777;
    font-size: 9px;
    line-height: 10px;
    text-align: center;
}
.ga-cor-percentual-status.falta { color: #c00; }
#produto .grade.ga-percentual-por-cor .ga-tamanhos-cabecalho,
#produto-sku .grade.ga-percentual-por-cor .ga-tamanhos-cabecalho {
    width: calc(100% - 126px) !important;
    margin-left: 126px !important;
}
</style>

<script>
(function () {
    "use strict";
    if (window.__gaModoPercentuaisCores) return;
    window.__gaModoPercentuaisCores = true;

    var modo = "tamanho";
    var areaAnterior = null;
    var agendado = false;

    function grade() {
        return document.querySelector("#produto-sku .grade, #produto .grade");
    }
    function area() {
        return document.querySelector(".ga-percentuais-editaveis");
    }
    function texto(elemento, valor) {
        if (elemento && elemento.textContent !== valor) elemento.textContent = valor;
    }
    function campos() {
        var g = grade();
        return g ? Array.from(g.querySelectorAll(".ga-cor-percentual")) : [];
    }
    function soma() {
        return campos().reduce(function (total, input) {
            return total + (parseInt(input.value, 10) || 0);
        }, 0);
    }
    function mensagem(valor, erro) {
        var a = area();
        if (!a) return;
        var resumo = a.querySelector(".ga-percentuais-total");
        texto(resumo, valor);
        if (resumo) resumo.classList.toggle("erro", Boolean(erro));
    }
    function atualizarSoma() {
        if (modo !== "cor") return;
        var total = soma();
        mensagem(total === 100 ? "Total: 100%" : "Faltam " + (100 - total) + "%", total !== 100);
        var a = area();
        var botao = a && a.querySelector(".ga-aplicar-percentuais");
        if (botao) botao.disabled = total !== 100;
    }
    function mudarModo(novo) {
        modo = novo;
        var a = area();
        var g = grade();
        if (!a || !g) return;
        a.classList.toggle("ga-modo-cores-percentual", modo === "cor");
        g.classList.toggle("ga-percentual-por-cor", modo === "cor");
        a.querySelectorAll(".ga-modo-percentuais button").forEach(function (botao) {
            var ativo = botao.dataset.modo === modo;
            botao.classList.toggle("ativo", ativo);
            botao.setAttribute("aria-pressed", String(ativo));
        });
        texto(a.querySelector(".ga-percentuais-titulo"), modo === "cor"
            ? "Informe as porcentagens ao lado das cores"
            : "Distribuição por tamanho e cores sortidas");
        texto(a.querySelector(".ga-limpar-percentuais"), modo === "cor"
            ? "Limpar porcentagens das cores" : "Limpar porcentagens");
        if (modo === "cor") atualizarSoma();
        else {
            var input = a.querySelector(".ga-percentual-input");
            if (input) input.dispatchEvent(new Event("input", { bubbles: true }));
        }
    }
    function instalar() {
        var g = grade();
        var a = area();
        if (!g || !a) return;

        g.querySelectorAll(".l").forEach(function (linha, indice) {
            var cor = linha.querySelector(".cor");
            if (!cor || cor.querySelector(".ga-cor-percentual-campo")) return;
            var label = document.createElement("label");
            label.className = "ga-cor-percentual-campo";
            label.innerHTML = '<span class="ga-cor-percentual-caixa">' +
                '<input type="number" class="ga-cor-percentual" min="0" max="100" step="1" value="0">' +
                '<span>%</span></span><small class="ga-cor-percentual-status"></small>';
            var input = label.querySelector("input");
            input.setAttribute("aria-label", "Porcentagem da cor " + (indice + 1));
            input.addEventListener("input", function () {
                var outros = soma() - (parseInt(input.value, 10) || 0);
                input.value = Math.max(0, Math.min(parseInt(input.value, 10) || 0, 100 - outros));
                atualizarSoma();
            });
            cor.appendChild(label);
        });

        if (!a.querySelector(".ga-modo-percentuais")) {
            var seletor = document.createElement("div");
            seletor.className = "ga-modo-percentuais";
            seletor.innerHTML = '<button type="button" data-modo="tamanho">Por tamanho</button>' +
                '<button type="button" data-modo="cor">Por cor</button>';
            seletor.querySelectorAll("button").forEach(function (botao) {
                botao.addEventListener("click", function () { mudarModo(botao.dataset.modo); });
            });
            a.insertBefore(seletor, a.firstChild);
        }
        g.classList.toggle("ga-percentual-por-cor", modo === "cor");
        if (areaAnterior !== a) {
            areaAnterior = a;
            mudarModo(modo);
        }
    }
    function quotas(total, porcentagens) {
        var resultado = porcentagens.map(function (p) { return Math.floor(total * p / 100); });
        var ordem = porcentagens.map(function (p, indice) {
            return { indice: indice, fracao: total * p / 100 - resultado[indice] };
        }).sort(function (a, b) { return b.fracao - a.fracao; });
        var restante = total - resultado.reduce(function (a, b) { return a + b; }, 0);
        for (var i = 0; i < restante; i++) resultado[ordem[i % ordem.length].indice]++;
        return resultado;
    }
    function celulas(linha) {
        var cor = linha.querySelector(".cor_primaria");
        if (cor && cor.classList.contains("sem_estoque")) return [];
        return Array.from(linha.querySelectorAll(".vars .it")).map(function (item) {
            var input = item.querySelector('input:not([type="hidden"])');
            if (!input || input.disabled || item.querySelector(".aviseme")) return null;
            var limite = input.getAttribute("max") || input.getAttribute("data-max") || input.getAttribute("data-estoque");
            var maximo = limite === null ? NaN : Number(limite);
            return { input: input, maximo: Number.isFinite(maximo) && maximo >= 0 ? Math.floor(maximo) : Infinity, quantidade: 0 };
        }).filter(Boolean);
    }
    function sortear(lista, quantidade) {
        var restante = quantidade;
        var ativos = lista.filter(function (item) { return item.maximo > 0; });
        while (restante > 0 && ativos.length) {
            for (var i = ativos.length - 1; i > 0; i--) {
                var j = Math.floor(Math.random() * (i + 1));
                var auxiliar = ativos[i]; ativos[i] = ativos[j]; ativos[j] = auxiliar;
            }
            var lote = Math.max(1, Math.floor(restante / ativos.length));
            ativos.forEach(function (item) {
                var adicionar = Math.min(lote, item.maximo - item.quantidade, restante);
                item.quantidade += adicionar;
                restante -= adicionar;
            });
            ativos = ativos.filter(function (item) { return item.quantidade < item.maximo; });
        }
        return quantidade - restante;
    }
    function alterarInput(input, valor) {
        if (String(input.value) === String(valor)) return;
        input.value = valor;
        input.dispatchEvent(new Event("input", { bubbles: true }));
        input.dispatchEvent(new Event("change", { bubbles: true }));
    }
    function distribuirCores() {
        var a = area();
        var g = grade();
        var painel = a && a.closest(".ga-grade-acoes");
        var totalInput = painel && painel.querySelector(".ga-sortidos-qtd");
        var total = totalInput ? Number(totalInput.value) : 0;
        if (!Number.isSafeInteger(total) || total <= 0) {
            mensagem("Informe uma quantidade inteira de peças.", true);
            return;
        }
        if (soma() !== 100) { atualizarSoma(); return; }
        var linhas = Array.from(g.querySelectorAll(".l")).filter(function (linha) {
            return linha.querySelector(".ga-cor-percentual");
        });
        var quantidades = quotas(total, linhas.map(function (linha) {
            return Number(linha.querySelector(".ga-cor-percentual").value) || 0;
        }));
        g.querySelectorAll('.vars .it input:not([type="hidden"])').forEach(function (input) {
            if (!input.disabled) alterarInput(input, 0);
        });
        var faltas = 0;
        linhas.forEach(function (linha, indice) {
            var lista = celulas(linha);
            var preenchidas = sortear(lista, quantidades[indice]);
            lista.forEach(function (item) { alterarInput(item.input, item.quantidade); });
            var falta = quantidades[indice] - preenchidas;
            faltas += falta;
            var status = linha.querySelector(".ga-cor-percentual-status");
            texto(status, falta > 0 ? "Faltam " + falta + " peças" : preenchidas + " peças");
            if (status) status.classList.toggle("falta", falta > 0);
        });
        mensagem(faltas ? "Faltam " + faltas + " peças no total" : "Distribuição concluída", faltas > 0);
    }
    document.addEventListener("click", function (evento) {
        if (modo !== "cor" || !(evento.target instanceof Element)) return;
        var botao = evento.target.closest(".ga-aplicar-percentuais, .ga-limpar-percentuais");
        if (!botao || !botao.closest(".ga-percentuais-editaveis")) return;
        evento.preventDefault();
        evento.stopImmediatePropagation();
        if (botao.classList.contains("ga-aplicar-percentuais")) distribuirCores();
        else {
            campos().forEach(function (input) { input.value = 0; });
            var g = grade();
            if (g) g.querySelectorAll(".ga-cor-percentual-status").forEach(function (status) {
                texto(status, ""); status.classList.remove("falta");
            });
            atualizarSoma();
        }
    }, true);
    function agendar() {
        if (agendado) return;
        agendado = true;
        setTimeout(function () { agendado = false; instalar(); }, 100);
    }
    function iniciar() {
        instalar();
        new MutationObserver(agendar).observe(document.body, { childList: true, subtree: true });
    }
    if (document.readyState === "loading") document.addEventListener("DOMContentLoaded", iniciar, { once: true });
    else iniciar();
})();
</script>
<script>
(function () {
    if (window.__gaRemoverMarcadores) return;
    window.__gaRemoverMarcadores = true;

    function limparMarcadores() {
        document.querySelectorAll(
            ".ga-percentual-status, .ga-cor-percentual-status"
        ).forEach(function (status) {
            var texto = status.textContent.trim();

            if (/^[?¿???-]+$/.test(texto)) {
                status.textContent = "";
            }
        });
    }

    function iniciar() {
        limparMarcadores();

        new MutationObserver(limparMarcadores).observe(
            document.body,
            {
                childList: true,
                subtree: true
            }
        );
    }

    if (document.readyState === "loading") {
        document.addEventListener(
            "DOMContentLoaded",
            iniciar,
            { once: true }
        );
    } else {
        iniciar();
    }
})();
</script>
<style>
.ga-qtd-cores { display: none !important; }
.ga-modo-cores-percentual .ga-qtd-cores {
    display: block !important;
    margin: 8px 0 16px;
}
.ga-qtd-cores-ativar {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 7px;
    margin-bottom: 12px;
    font-size: 12px;
    color: #333;
}
.ga-qtd-cores-ativar input {
    width: 16px !important;
    height: 16px !important;
    min-height: 0 !important;
    margin: 0 !important;
    accent-color: #299b16;
}
.ga-qtd-cores-config[hidden] { display: none !important; }
.ga-qtd-cores-campos {
    display: grid;
    grid-template-columns: repeat(var(--ga-total-tamanhos, 4), minmax(0, 1fr));
    gap: 8px;
}
.ga-qtd-tamanho-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    min-width: 0;
}
.ga-qtd-tamanho-nome {
    margin-bottom: 5px;
    color: #222;
    font-size: 13px;
    font-weight: 600;
}
.ga-qtd-tamanho-input {
    width: 72px !important;
    max-width: 100% !important;
    height: 40px !important;
    padding: 0 4px !important;
    border: 1px solid #d6d6d6 !important;
    border-radius: 5px !important;
    background: #fff !important;
    color: #222 !important;
    text-align: center !important;
    font-size: 13px !important;
}
.ga-qtd-tamanho-status {
    min-height: 14px;
    margin-top: 4px;
    font-size: 10px;
    line-height: 12px;
    color: #777;
    text-align: center;
}
.ga-qtd-tamanho-status.falta { color: #c00; }
.ga-qtd-cores-total {
    margin-top: 7px;
    color: #555;
    font-size: 11px;
    text-align: center;
}
.ga-qtd-cores-ajuda {
    margin: 5px 0 0;
    color: #777;
    font-size: 11px;
    line-height: 15px;
    text-align: center;
}
</style>

<script>
(function () {
    "use strict";
    if (window.__gaCoresComQtdTamanhos) return;
    window.__gaCoresComQtdTamanhos = true;
    var pendente = false;

    function obterGrade() {
        return document.querySelector("#produto-sku .grade, #produto .grade");
    }
    function obterArea() {
        return document.querySelector(".ga-percentuais-editaveis");
    }
    function escrever(elemento, texto) {
        if (elemento && elemento.textContent !== texto) elemento.textContent = texto;
    }
    function obterTamanhos(grade) {
        var primeiraLinha = grade.querySelector(".l");
        if (!primeiraLinha) return [];
        var itens = Array.from(primeiraLinha.querySelectorAll(".vars .it"));
        var cabecalho = Array.from(grade.querySelectorAll(".ga-tamanhos-cabecalho span"));
        if (!cabecalho.length) cabecalho = Array.from(document.querySelectorAll(".ga-tamanhos-cabecalho span"));
        return itens.map(function (item, indice) {
            var elemento = item.querySelector(".t p:first-child, .t, [data-tamanho], [data-size]");
            var nome = cabecalho.length === itens.length ? cabecalho[indice].textContent.trim() : "";
            if (!nome && elemento) nome = elemento.getAttribute("data-tamanho") || elemento.getAttribute("data-size") || elemento.textContent.trim();
            return nome || "Tamanho " + (indice + 1);
        });
    }
    function atualizarTotal(painel, sincronizar) {
        var inputs = Array.from(painel.querySelectorAll(".ga-qtd-tamanho-input"));
        var total = inputs.reduce(function (soma, input) { return soma + (Number(input.value) || 0); }, 0);
        escrever(painel.querySelector(".ga-qtd-cores-total"), "Total por tamanho: " + total + " peças");
        if (!sincronizar || !Number.isSafeInteger(total)) return;
        var principal = painel.closest(".ga-grade-acoes");
        var campoTotal = principal && principal.querySelector(".ga-sortidos-qtd");
        if (campoTotal && Number(campoTotal.value) !== total) {
            campoTotal.value = total;
            campoTotal.dispatchEvent(new Event("input", { bubbles: true }));
            campoTotal.dispatchEvent(new Event("change", { bubbles: true }));
        }
    }
    function instalar() {
        var grade = obterGrade();
        var area = obterArea();
        if (!grade || !area) return;
        var tamanhos = obterTamanhos(grade);
        if (!tamanhos.length) return;
        var assinatura = tamanhos.join("|");
        var painel = area.querySelector(".ga-qtd-cores");
        if (painel && painel.dataset.tamanhos === assinatura) return;
        if (painel) painel.remove();

        painel = document.createElement("div");
        painel.className = "ga-qtd-cores";
        painel.dataset.tamanhos = assinatura;
        painel.innerHTML = '<label class="ga-qtd-cores-ativar">' +
            '<input type="checkbox" class="ga-fixar-qtd-tamanhos" checked>' +
            '<span>Definir peças por tamanho</span></label>' +
            '<div class="ga-qtd-cores-config"><div class="ga-qtd-cores-campos"></div>' +
            '<div class="ga-qtd-cores-total"></div>' +
            '<p class="ga-qtd-cores-ajuda">Informe as peças por tamanho e as porcentagens nas cores.</p></div>';
        var campos = painel.querySelector(".ga-qtd-cores-campos");
        campos.style.setProperty("--ga-total-tamanhos", Math.min(tamanhos.length, 4));
        tamanhos.forEach(function (nome, indice) {
            var card = document.createElement("label");
            card.className = "ga-qtd-tamanho-card";
            var titulo = document.createElement("span");
            titulo.className = "ga-qtd-tamanho-nome";
            titulo.textContent = nome;
            var input = document.createElement("input");
            input.type = "number";
            input.className = "ga-qtd-tamanho-input";
            input.min = "0"; input.step = "1"; input.value = "0";
            input.dataset.indice = indice;
            input.setAttribute("aria-label", "Peças do tamanho " + nome);
            var status = document.createElement("small");
            status.className = "ga-qtd-tamanho-status";
            card.appendChild(titulo); card.appendChild(input); card.appendChild(status);
            campos.appendChild(card);
            input.addEventListener("input", function () {
                var valor = Math.floor(Number(input.value));
                input.value = Number.isSafeInteger(valor) && valor >= 0 ? valor : 0;
                atualizarTotal(painel, true);
            });
        });
        painel.querySelector(".ga-fixar-qtd-tamanhos").addEventListener("change", function (evento) {
            painel.querySelector(".ga-qtd-cores-config").hidden = !evento.target.checked;
        });
        var tituloArea = area.querySelector(".ga-percentuais-titulo");
        if (tituloArea) area.insertBefore(painel, tituloArea);
        else area.appendChild(painel);
        atualizarTotal(painel, false);
    }
    function quotas(total, percentuais) {
        var resultado = percentuais.map(function (p) { return Math.floor(total * p / 100); });
        var ordem = percentuais.map(function (p, indice) {
            return { indice: indice, fracao: total * p / 100 - resultado[indice] };
        }).sort(function (a, b) { return b.fracao - a.fracao; });
        var restante = total - resultado.reduce(function (a, b) { return a + b; }, 0);
        for (var i = 0; i < restante; i++) resultado[ordem[i % ordem.length].indice]++;
        return resultado;
    }
    function celula(linha, indice) {
        var cor = linha.querySelector(".cor_primaria");
        var item = linha.querySelectorAll(".vars .it")[indice];
        if (!item || item.querySelector(".aviseme") || (cor && cor.classList.contains("sem_estoque"))) return null;
        var input = item.querySelector('input:not([type="hidden"])');
        if (!input || input.disabled) return null;
        var limite = input.getAttribute("max") || input.getAttribute("data-max") || input.getAttribute("data-estoque");
        var maximo = limite === null ? NaN : Number(limite);
        return { input: input, maximo: Number.isFinite(maximo) && maximo >= 0 ? Math.floor(maximo) : Infinity };
    }
    function resumo(area, texto, erro) {
        var elemento = area.querySelector(".ga-percentuais-total");
        escrever(elemento, texto);
        if (elemento) elemento.classList.toggle("erro", Boolean(erro));
    }
    function distribuir(area, grade, painel) {
        var inputs = Array.from(painel.querySelectorAll(".ga-qtd-tamanho-input"));
        var quantidades = inputs.map(function (input) { return Number(input.value); });
        if (quantidades.some(function (q) { return !Number.isSafeInteger(q) || q < 0; })) {
            resumo(area, "Informe quantidades inteiras por tamanho.", true); return;
        }
        var total = quantidades.reduce(function (a, b) { return a + b; }, 0);
        if (!Number.isSafeInteger(total) || total <= 0) {
            resumo(area, "Informe as peças por tamanho.", true); return;
        }
        var principal = area.closest(".ga-grade-acoes");
        var campoTotal = principal && principal.querySelector(".ga-sortidos-qtd");
        if (campoTotal && Number(campoTotal.value) !== total) {
            resumo(area, "Os tamanhos somam " + total + "; ajuste a quantidade total.", true); return;
        }
        var linhas = Array.from(grade.querySelectorAll(".l")).filter(function (linha) {
            return linha.querySelector(".ga-cor-percentual");
        });
        var percentuais = linhas.map(function (linha) { return Number(linha.querySelector(".ga-cor-percentual").value); });
        if (!percentuais.length || percentuais.some(function (p) { return !Number.isInteger(p) || p < 0 || p > 100; }) ||
            percentuais.reduce(function (a, b) { return a + b; }, 0) !== 100) {
            resumo(area, "As cores precisam somar 100%.", true); return;
        }

        var plano = new Map();
        var corPreenchida = linhas.map(function () { return 0; });
        var corFaltante = linhas.map(function () { return 0; });
        var tamanhoPreenchido = quantidades.map(function () { return 0; });
        var tamanhoFaltante = quantidades.map(function () { return 0; });
        quantidades.forEach(function (quantidade, indiceTamanho) {
            var porCor = quotas(quantidade, percentuais);
            linhas.forEach(function (linha, indiceCor) {
                var destino = celula(linha, indiceTamanho);
                var desejada = porCor[indiceCor];
                var preenchida = destino ? Math.min(desejada, destino.maximo) : 0;
                if (destino) plano.set(destino.input, preenchida);
                corPreenchida[indiceCor] += preenchida;
                corFaltante[indiceCor] += desejada - preenchida;
                tamanhoPreenchido[indiceTamanho] += preenchida;
                tamanhoFaltante[indiceTamanho] += desejada - preenchida;
            });
        });
        var alterados = [];
        grade.querySelectorAll('.vars .it input:not([type="hidden"])').forEach(function (input) {
            if (input.disabled) return;
            var valor = plano.get(input) || 0;
            if (String(input.value) !== String(valor)) {
                input.value = valor;
                alterados.push(input);
            }
        });
        alterados.forEach(function (input) {
            input.dispatchEvent(new Event("input", { bubbles: true }));
            input.dispatchEvent(new Event("change", { bubbles: true }));
        });
        linhas.forEach(function (linha, indice) {
            var status = linha.querySelector(".ga-cor-percentual-status");
            escrever(status, corFaltante[indice] ? "Faltam " + corFaltante[indice] + " peças" : corPreenchida[indice] + " peças");
            if (status) status.classList.toggle("falta", corFaltante[indice] > 0);
        });
        inputs.forEach(function (input, indice) {
            var status = input.parentElement.querySelector(".ga-qtd-tamanho-status");
            escrever(status, tamanhoFaltante[indice] ? "Faltam " + tamanhoFaltante[indice] + " peças" : tamanhoPreenchido[indice] + " peças");
            if (status) status.classList.toggle("falta", tamanhoFaltante[indice] > 0);
        });
        var faltas = tamanhoFaltante.reduce(function (a, b) { return a + b; }, 0);
        resumo(area, faltas ? "Faltam " + faltas + " peças no total" : "Distribuição concluída", faltas > 0);
    }
    /* Captura antes do distribuidor antigo, apenas quando esta opcao estiver ativa. */
    window.addEventListener("click", function (evento) {
        if (!(evento.target instanceof Element)) return;
        var botao = evento.target.closest(".ga-aplicar-percentuais, .ga-limpar-percentuais");
        var area = botao && botao.closest(".ga-percentuais-editaveis");
        var grade = obterGrade();
        var painel = area && area.querySelector(".ga-qtd-cores");
        var ativar = painel && painel.querySelector(".ga-fixar-qtd-tamanhos");
        if (!area || !grade || !grade.classList.contains("ga-percentual-por-cor") || !ativar || !ativar.checked) return;
        if (botao.classList.contains("ga-limpar-percentuais")) {
            painel.querySelectorAll(".ga-qtd-tamanho-status").forEach(function (status) {
                escrever(status, ""); status.classList.remove("falta");
            });
            return;
        }
        evento.preventDefault();
        evento.stopImmediatePropagation();
        distribuir(area, grade, painel);
    }, true);
    function agendar() {
        if (pendente) return;
        pendente = true;
        setTimeout(function () { pendente = false; instalar(); }, 100);
    }
    function iniciar() {
        instalar();
        new MutationObserver(agendar).observe(document.body, { childList: true, subtree: true });
    }
    if (document.readyState === "loading") document.addEventListener("DOMContentLoaded", iniciar, { once: true });
    else iniciar();
})();
</script>
<script>
(function () {
    "use strict";
    if (window.__gaCoresTamanhoDoSortido) return;
    window.__gaCoresTamanhoDoSortido = true;

    function escrever(elemento, texto) {
        if (elemento && elemento.textContent !== texto) elemento.textContent = texto;
    }
    function mensagem(area, texto, erro) {
        var resumo = area.querySelector(".ga-percentuais-total");
        escrever(resumo, texto);
        if (resumo) resumo.classList.toggle("erro", Boolean(erro));
    }
    function obterTamanhos(grade) {
        var primeira = grade.querySelector(".l");
        if (!primeira) return [];
        var itens = Array.from(primeira.querySelectorAll(".vars .it"));
        var cabecalho = Array.from(grade.querySelectorAll(".ga-tamanhos-cabecalho span"));
        if (!cabecalho.length) cabecalho = Array.from(document.querySelectorAll(".ga-tamanhos-cabecalho span"));
        return itens.map(function (item, indice) {
            var texto = cabecalho.length === itens.length ? cabecalho[indice].textContent.trim() : "";
            var elemento = item.querySelector(".t p:first-child, .t, [data-tamanho], [data-size]");
            if (!texto && elemento) texto = elemento.getAttribute("data-tamanho") || elemento.getAttribute("data-size") || elemento.textContent.trim();
            return texto;
        });
    }
    function indiceSelecionado(select, grade) {
        if (!select || select.value === "") return -1;
        var tamanhos = obterTamanhos(grade);
        var indice = Number(select.value);
        if (Number.isInteger(indice) && indice >= 0 && indice < tamanhos.length) return indice;
        var opcao = select.options[select.selectedIndex];
        var nome = opcao ? opcao.textContent.trim().toUpperCase() : "";
        return tamanhos.findIndex(function (tamanho) { return tamanho.toUpperCase() === nome; });
    }
    function quotas(total, percentuais) {
        var resultado = percentuais.map(function (p) { return Math.floor(total * p / 100); });
        var ordem = percentuais.map(function (p, indice) {
            return { indice: indice, fracao: total * p / 100 - resultado[indice] };
        }).sort(function (a, b) { return b.fracao - a.fracao; });
        var restante = total - resultado.reduce(function (a, b) { return a + b; }, 0);
        for (var i = 0; i < restante; i++) resultado[ordem[i % ordem.length].indice]++;
        return resultado;
    }
    function obterDestino(linha, indice) {
        var item = linha.querySelectorAll(".vars .it")[indice];
        var input = item && item.querySelector('input:not([type="hidden"])');
        if (!input || input.disabled) return { input: null, maximo: 0 };
        var cor = linha.querySelector(".cor_primaria");
        if (item.querySelector(".aviseme") || (cor && cor.classList.contains("sem_estoque"))) {
            return { input: input, maximo: 0 };
        }
        var limite = input.getAttribute("max") || input.getAttribute("data-max") || input.getAttribute("data-estoque");
        var maximo = limite === null ? NaN : Number(limite);
        return { input: input, maximo: Number.isFinite(maximo) && maximo >= 0 ? Math.floor(maximo) : Infinity };
    }
    function sortearTamanhos(destinos, quantidade) {
        destinos.forEach(function (destino) { destino.quantidade = 0; });
        var restantes = quantidade;
        var ativos = destinos.filter(function (destino) { return destino.input && destino.maximo > 0; });
        while (restantes > 0 && ativos.length) {
            for (var i = ativos.length - 1; i > 0; i--) {
                var j = Math.floor(Math.random() * (i + 1));
                var auxiliar = ativos[i]; ativos[i] = ativos[j]; ativos[j] = auxiliar;
            }
            var lote = Math.max(1, Math.floor(restantes / ativos.length));
            ativos.forEach(function (destino) {
                var adicionar = Math.min(lote, destino.maximo - destino.quantidade, restantes);
                destino.quantidade += adicionar;
                restantes -= adicionar;
            });
            ativos = ativos.filter(function (destino) { return destino.quantidade < destino.maximo; });
        }
        return quantidade - restantes;
    }
    function distribuir(area, painel, grade) {
        var campo = painel.querySelector(".ga-sortidos-qtd");
        var select = painel.querySelector(".ga-sortidos-tamanho");
        var total = campo ? Number(campo.value) : 0;
        if (!Number.isSafeInteger(total) || total <= 0) {
            mensagem(area, "Informe a quantidade de peças.", true); return;
        }
        var indice = indiceSelecionado(select, grade);
        var aleatorio = !select || select.value === "";
        if (!aleatorio && indice < 0) {
            mensagem(area, "O tamanho escolhido não foi encontrado.", true); return;
        }
        var linhas = Array.from(grade.querySelectorAll(".l")).filter(function (linha) {
            return linha.querySelector(".ga-cor-percentual");
        });
        var percentuais = linhas.map(function (linha) { return Number(linha.querySelector(".ga-cor-percentual").value); });
        var soma = percentuais.reduce(function (a, b) { return a + b; }, 0);
        if (!percentuais.length || percentuais.some(function (p) { return !Number.isInteger(p) || p < 0 || p > 100; }) || soma !== 100) {
            mensagem(area, soma >= 0 && soma < 100 ? "Faltam " + (100 - soma) + "% nas cores" : "As cores precisam somar 100%.", true);
            return;
        }
        var desejadas = quotas(total, percentuais);
        var faltas = 0;
        var alterados = [];
        linhas.forEach(function (linha, indiceCor) {
            var quantidade;
            var destinos;
            if (aleatorio) {
                destinos = Array.from(linha.querySelectorAll(".vars .it")).map(function (item, indiceTamanho) {
                    return obterDestino(linha, indiceTamanho);
                });
                quantidade = sortearTamanhos(destinos, desejadas[indiceCor]);
            } else {
                var destino = obterDestino(linha, indice);
                quantidade = Math.min(desejadas[indiceCor], destino.maximo);
                destino.quantidade = quantidade;
                destinos = [destino];
            }
            var falta = desejadas[indiceCor] - quantidade;
            faltas += falta;
            destinos.forEach(function (destino) {
                if (destino.input && String(destino.input.value) !== String(destino.quantidade)) {
                    destino.input.value = destino.quantidade;
                    alterados.push(destino.input);
                }
            });
            var status = linha.querySelector(".ga-cor-percentual-status");
            escrever(status, falta ? "Faltam " + falta + " peças" : quantidade + " peças");
            if (status) status.classList.toggle("falta", falta > 0);
        });
        alterados.forEach(function (input) {
            input.dispatchEvent(new Event("input", { bubbles: true }));
            input.dispatchEvent(new Event("change", { bubbles: true }));
        });
        var opcao = select && select.options[select.selectedIndex];
        var nome = aleatorio ? "tamanhos aleatórios" : (opcao ? opcao.textContent.trim() : "");
        mensagem(area, faltas ? "Faltam " + faltas + " peças em " + nome : "Distribuição concluída: " + total + " peças em " + nome, faltas > 0);
        var mensagemAntiga = painel.querySelector(".ga-grade-mensagem, .ga-sortidos-mensagem");
               if (mensagemAntiga) {
            escrever(mensagemAntiga, "");
            mensagemAntiga.classList.remove("erro");
        }
    }
    window.addEventListener("click", function (evento) {
        if (!(evento.target instanceof Element)) return;
        var botao = evento.target.closest(".ga-sortidos-btn, .ga-aplicar-percentuais");
        var painel = botao && botao.closest(".ga-grade-acoes");
        if (!painel) return;
        var area = painel.querySelector(".ga-percentuais-editaveis");
        var grade = document.querySelector("#produto-sku .grade, #produto .grade");
        var checkbox = area && area.querySelector(".ga-fixar-qtd-tamanhos");
        if (!grade || !grade.classList.contains("ga-percentual-por-cor") || !checkbox || checkbox.checked) return;
        evento.preventDefault();
        evento.stopImmediatePropagation();
        distribuir(area, painel, grade);
    }, true);
})();
</script>
<style>
#produto-sku.ga-funcoes-grade .ga-grade-acoes button.ga-tutorial-grade,
#produto.ga-funcoes-grade .ga-grade-acoes button.ga-tutorial-grade,
button.ga-tutorial-grade {
    display: inline-flex !important;
    align-items: center !important;
    justify-content: center !important;
    gap: 7px !important;
    min-height: 38px !important;
    padding: 0 14px !important;
    border: 1px solid #d4d4d4 !important;
    border-radius: 5px !important;
    background: #fff !important;
    color: #333 !important;
    font-size: 12px !important;
    text-indent: 0 !important;
    cursor: pointer !important;
}
button.ga-tutorial-grade svg {
    display: block;
    width: 15px;
    height: 15px;
    color: #299b16;
    flex: 0 0 15px;
}
#produto-sku.ga-funcoes-grade .ga-grade-acoes button.ga-tutorial-grade:hover,
#produto.ga-funcoes-grade .ga-grade-acoes button.ga-tutorial-grade:hover {
    background: #f4f8f2 !important;
    border-color: #299b16 !important;
}
#ga-tutorial-modal {
    width: min(860px, calc(100% - 24px));
    max-width: none;
    max-height: calc(100vh - 24px);
    margin: auto;
    padding: 0;
    border: 0;
    border-radius: 10px;
    background: #fff;
    color: #222;
    overflow: auto;
    box-shadow: 0 20px 70px rgba(0, 0, 0, .3);
    box-sizing: border-box;
}
#ga-tutorial-modal::backdrop { background: rgba(0, 0, 0, .65); }
#ga-tutorial-modal .ga-tutorial-cabecalho {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    padding: 14px 16px;
}
#ga-tutorial-modal .ga-tutorial-titulo {
    margin: 0;
    font-size: 15px;
    font-weight: 600;
}
#ga-tutorial-modal .ga-tutorial-fechar {
    padding: 7px 12px !important;
    border: 1px solid #ddd !important;
    border-radius: 5px !important;
    background: #fff !important;
    color: #333 !important;
    font-size: 12px !important;
    text-indent: 0 !important;
    cursor: pointer !important;
}
#ga-tutorial-modal .ga-tutorial-video {
    position: relative;
    width: 100%;
    aspect-ratio: 16 / 9;
    background: #111;
}
#ga-tutorial-modal .ga-tutorial-video iframe {
    position: absolute;
    inset: 0;
    display: block;
    width: 100%;
    height: 100%;
    border: 0;
}
#ga-tutorial-modal .ga-tutorial-rodape {
    padding: 12px 16px;
    font-size: 12px;
    text-align: center;
}
#ga-tutorial-modal .ga-tutorial-rodape a {
    color: #238912;
    text-decoration: underline;
}
</style>
<script>
(function () {
    "use strict";
    if (window.__gaTutorialDaGrade) return;
    window.__gaTutorialDaGrade = true;

    var URL_VIDEO = "https://www.youtube.com/watch?v=c4C061bSpVo";
    var URL_EMBED = "https://www.youtube-nocookie.com/embed/c4C061bSpVo?rel=0";
    var modal = null;
    var focoAnterior = null;
    var pendente = false;

    function criarModal() {
        if (modal) return modal;
        modal = document.createElement("dialog");
        modal.id = "ga-tutorial-modal";
        modal.setAttribute("aria-labelledby", "ga-tutorial-titulo");
        modal.innerHTML = '<div class="ga-tutorial-cabecalho">' +
            '<h2 class="ga-tutorial-titulo" id="ga-tutorial-titulo">Como usar a grade</h2>' +
            '<button type="button" class="ga-tutorial-fechar" autofocus>Fechar</button></div>' +
            '<div class="ga-tutorial-video"></div>' +
            '<div class="ga-tutorial-rodape"><a href="' + URL_VIDEO + '" ' +
            'target="_blank" rel="noopener noreferrer">Abrir o tutorial no YouTube</a></div>';
        modal.querySelector(".ga-tutorial-fechar").addEventListener("click", function () {
            modal.close();
        });
        modal.addEventListener("close", function () {
            modal.querySelector(".ga-tutorial-video").replaceChildren();
            if (focoAnterior && focoAnterior.isConnected) focoAnterior.focus();
        });
        modal.addEventListener("click", function (evento) {
            if (evento.target !== modal) return;
            var retangulo = modal.getBoundingClientRect();
            if (evento.clientX < retangulo.left || evento.clientX > retangulo.right ||
                evento.clientY < retangulo.top || evento.clientY > retangulo.bottom) modal.close();
        });
        document.body.appendChild(modal);
        return modal;
    }
    function abrirTutorial(botao) {
        var janela = criarModal();
        if (typeof janela.showModal !== "function") {
            window.open(URL_VIDEO, "_blank", "noopener,noreferrer");
            return;
        }
        if (janela.open) return;
        focoAnterior = botao;
        var iframe = document.createElement("iframe");
        iframe.src = URL_EMBED;
        iframe.title = "Tutorial da grade de compras Ilumine";
        iframe.allow = "accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture; web-share";
        iframe.allowFullscreen = true;
        iframe.referrerPolicy = "strict-origin-when-cross-origin";
        janela.querySelector(".ga-tutorial-video").replaceChildren(iframe);
        janela.showModal();
    }
    function instalar() {
        document.querySelectorAll(".ga-grade-acoes").forEach(function (painel) {
            if (painel.querySelector(".ga-tutorial-grade")) return;
            var sortidos = painel.querySelector(".ga-sortidos-btn");
            if (!sortidos) return;
            var botao = document.createElement("button");
            botao.type = "button";
            botao.className = "ga-tutorial-grade";
            botao.setAttribute("aria-haspopup", "dialog");
            botao.innerHTML = '<svg viewBox="0 0 20 20" aria-hidden="true" focusable="false">' +
                '<path d="M6 3L17 10L6 17Z" fill="currentColor"></path></svg>' +
                '<span>Como usar a grade</span>';
            botao.addEventListener("click", function () { abrirTutorial(botao); });
            sortidos.insertAdjacentElement("afterend", botao);
        });
    }
    function agendar() {
        if (pendente) return;
        pendente = true;
        setTimeout(function () { pendente = false; instalar(); }, 100);
    }
    function iniciar() {
        instalar();
        new MutationObserver(agendar).observe(document.body, { childList: true, subtree: true });
    }
    if (document.readyState === "loading") document.addEventListener("DOMContentLoaded", iniciar, { once: true });
    else iniciar();
})();
</script>
<style>
.ga-tutorial-ajuda {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 10px;

    width: 100%;
    margin-bottom: 20px;
    padding: 12px 14px;

    border: 1px solid #e5e5e5;
    border-radius: 6px;
    background: #f8f8f8;

    box-sizing: border-box;
}

.ga-tutorial-ajuda-texto {
    color: #666;
    font-size: 12px;
}

@media (max-width: 480px) {
    .ga-tutorial-ajuda {
        justify-content: center;
        text-align: center;
    }
}
</style>

<script>
(function () {
    if (window.__gaSepararTutorial) return;
    window.__gaSepararTutorial = true;

    function reposicionar() {
        document.querySelectorAll(
            ".ga-grade-acoes"
        ).forEach(function (painel) {
            var botao = painel.querySelector(
                ".ga-tutorial-grade"
            );

            if (
                !botao ||
                botao.closest(".ga-tutorial-ajuda")
            ) {
                return;
            }

            var faixa = document.createElement("div");
            faixa.className = "ga-tutorial-ajuda";

            var texto = document.createElement("span");
            texto.className = "ga-tutorial-ajuda-texto";
            texto.textContent = "Precisa de ajuda?";

            faixa.appendChild(texto);
            faixa.appendChild(botao);

            painel.insertBefore(
                faixa,
                painel.firstChild
            );
        });
    }

    function iniciar() {
        reposicionar();

        new MutationObserver(reposicionar).observe(
    	        document.body,
            { childList: true, subtree: true }
        );
    }

    if (document.readyState === "loading") {
        document.addEventListener(
            "DOMContentLoaded",
            iniciar,
            { once: true }
        );
    } else {
        iniciar();
    }
})();
</script>
<style>
#produto .ga-grade-acoes .ga-botoes-limpeza,
#produto-sku .ga-grade-acoes .ga-botoes-limpeza {
  display: grid !important;
  grid-template-columns: repeat(2, minmax(0, 1fr)) !important;
  gap: 12px !important;
  width: 100% !important;
  margin: 16px 0 0 !important;
}

#produto .ga-grade-acoes .ga-botoes-limpeza > button,
#produto-sku .ga-grade-acoes .ga-botoes-limpeza > button {
  display: flex !important;
  align-items: center !important;
  justify-content: center !important;
  width: 100% !important;
  min-width: 0 !important;
  min-height: 48px !important;
  height: auto !important;
  margin: 0 !important;
  padding: 10px 8px !important;
  box-sizing: border-box !important;
  border: 1px solid #d4d4d4 !important;
  border-radius: 6px !important;
  background: #fff !important;
  color: #444 !important;
  font-family: inherit !important;
  font-size: 12px !important;
  font-weight: 500 !important;
  line-height: 1.35 !important;
  text-align: center !important;
  text-transform: none !important;
  white-space: normal !important;
  overflow-wrap: anywhere !important;
  cursor: pointer !important;
}

#produto .ga-grade-acoes .ga-botoes-limpeza > button:hover,
#produto-sku .ga-grade-acoes .ga-botoes-limpeza > button:hover {
  background: #f5f5f5 !important;
  border-color: #999 !important;
}

#produto .ga-grade-acoes .ga-botoes-limpeza > button:focus-visible,
#produto-sku .ga-grade-acoes .ga-botoes-limpeza > button:focus-visible {
  outline: 2px solid #299b16 !important;
  outline-offset: 3px !important;
}
</style>

<script>
(function () {
  "use strict";
  if (window.__gaBotoesLimpezaJuntos) return;
  window.__gaBotoesLimpezaJuntos = true;

  var guardados = new WeakMap();

  function organizar() {
    document.querySelectorAll(".ga-grade-acoes").forEach(function (painel) {
      var pecas = painel.querySelector(".ga-limpar-grade") || guardados.get(painel);
      if (pecas) guardados.set(painel, pecas);

      var area = painel.querySelector(".ga-percentuais-editaveis");
      var porcentagens = area && area.querySelector(".ga-limpar-percentuais");
      var distribuir = area && area.querySelector(".ga-aplicar-percentuais");
      if (!pecas || !porcentagens || !distribuir) return;

      var linha = area.querySelector(".ga-botoes-limpeza");
      if (!linha) {
        linha = document.createElement("div");
        linha.className = "ga-botoes-limpeza";
      }

      
      if (pecas.textContent !== "Limpar peças") pecas.textContent = "Limpar peças";
      if (pecas.parentNode !== linha) linha.appendChild(pecas);
      if (porcentagens.parentNode !== linha) linha.appendChild(porcentagens);
      if (distribuir.nextElementSibling !== linha) {
        distribuir.insertAdjacentElement("afterend", linha);
      }
    });
  }

  function iniciar() {
    organizar();
    new MutationObserver(organizar).observe(document.body, {
      childList: true,
      subtree: true
    });
  }

  if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", iniciar, { once: true });
  } else {
    iniciar();
  }
})();
</script>
<script>
(function () {
  if (window.__gaTextoBotaoDistribuicao) return;
  window.__gaTextoBotaoDistribuicao = true;

  function atualizarTexto() {
    document.querySelectorAll(".ga-percentuais-editaveis").forEach(function (area) {
      var botao = area.querySelector(".ga-aplicar-percentuais");
      if (!botao) return;

      var texto = area.classList.contains("ga-modo-cores-percentual")
        ? "Distribuir por cor"
        : "Distribuir por porcentagem";

      if (botao.textContent !== texto) {
        botao.textContent = texto;
      }
    });
  }

  function iniciar() {
    atualizarTexto();

    new MutationObserver(atualizarTexto).observe(document.body, {
      childList: true,
      subtree: true,
      attributes: true,
      attributeFilter: ["class"]
    });
  }

  if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", iniciar, { once: true });
  } else {
    iniciar();
  }
})();
</script>

