#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define TAM_FILA 5
#define TAM_PILHA 3

// --------------------------------------
// ESTRUTURA DA PEÇA
// --------------------------------------
typedef struct {
    char nome; // 'I', 'O', 'T', 'L'
    int id;    // ordem de criação
} Peca;

// --------------------------------------
// VARIÁVEIS GLOBAIS
// --------------------------------------
int proximoID = 0; // controla IDs únicos

// --------------------------------------
// GERA PEÇA AUTOMATICAMENTE
// --------------------------------------
Peca gerarPeca() {
    char tipos[] = {'I', 'O', 'T', 'L'};
    int indice = rand() % 4;

    Peca p;
    p.nome = tipos[indice];
    p.id = proximoID++;

    return p;
}

// --------------------------------------
// FILA CIRCULAR
// --------------------------------------
typedef struct {
    Peca itens[TAM_FILA];
    int frente;
    int tras;
    int quantidade;
} Fila;

void inicializarFila(Fila *f) {
    f->frente = 0;
    f->tras = -1;
    f->quantidade = 0;

    for (int i = 0; i < TAM_FILA; i++) {
        f->tras = (f->tras + 1) % TAM_FILA;
        f->itens[f->tras] = gerarPeca();
        f->quantidade++;
    }
}

int filaVazia(Fila *f) {
    return f->quantidade == 0;
}

int filaCheia(Fila *f) {
    return f->quantidade == TAM_FILA;
}

Peca removerFila(Fila *f) {
    Peca removido = f->itens[f->frente];
    f->frente = (f->frente + 1) % TAM_FILA;
    f->quantidade--;
    return removido;
}

void inserirFila(Fila *f, Peca p) {
    if (filaCheia(f)) return;
    f->tras = (f->tras + 1) % TAM_FILA;
    f->itens[f->tras] = p;
    f->quantidade++;
}

// --------------------------------------
// PILHA
// --------------------------------------
typedef struct {
    Peca itens[TAM_PILHA];
    int topo;
} Pilha;

void inicializarPilha(Pilha *p) {
    p->topo = -1;
}

int pilhaVazia(Pilha *p) {
    return p->topo == -1;
}

int pilhaCheia(Pilha *p) {
    return p->topo == TAM_PILHA - 1;
}

void push(Pilha *p, Peca x) {
    if (pilhaCheia(p)) return;
    p->itens[++p->topo] = x;
}

Peca pop(Pilha *p) {
    return p->itens[p->topo--];
}

// --------------------------------------
// EXIBIR ESTADO ATUAL
// --------------------------------------
void exibirEstado(Fila *f, Pilha *p) {
    printf("\n=====================================\n");
    printf("ESTADO ATUAL:\n");
    printf("Fila de peças: ");

    int index = f->frente;
    for (int i = 0; i < f->quantidade; i++) {
        Peca pc = f->itens[index];
        printf("[%c %d] ", pc.nome, pc.id);
        index = (index + 1) % TAM_FILA;
    }

    printf("\nPilha de reserva (topo → base): ");
    for (int i = p->topo; i >= 0; i--) {
        printf("[%c %d] ", p->itens[i].nome, p->itens[i].id);
    }

    printf("\n=====================================\n");
}

// --------------------------------------
// TROCA DE TRÊS PEÇAS ENTRE FILA E PILHA
// --------------------------------------
void trocaTres(Fila *f, Pilha *p) {
    if (p->topo < 2 || f->quantidade < 3) {
        printf("Não foi possível realizar a troca múltipla.\n");
        return;
    }

    // remover 3 da fila
    Peca f1 = removerFila(f);
    Peca f2 = removerFila(f);
    Peca f3 = removerFila(f);

    // remover 3 da pilha
    Peca p1 = pop(p);
    Peca p2 = pop(p);
    Peca p3 = pop(p);

    // trocar
    push(p, f1);
    push(p, f2);
    push(p, f3);

    inserirFila(f, p1);
    inserirFila(f, p2);
    inserirFila(f, p3);

    printf("Troca de três peças realizada.\n");
}

// --------------------------------------
// MENU PRINCIPAL
// --------------------------------------
int main() {
    srand(time(NULL));

    Fila fila;
    Pilha pilha;
    inicializarFila(&fila);
    inicializarPilha(&pilha);

    int opcao;

    do {
        exibirEstado(&fila, &pilha);

        printf("\nAÇÕES DISPONÍVEIS:\n");
        printf("1 - Jogar peça da frente da fila\n");
        printf("2 - Enviar peça da fila para a pilha\n");
        printf("3 - Usar peça da pilha\n");
        printf("4 - Trocar peça da frente com o topo da pilha\n");
        printf("5 - Trocar os 3 primeiros da fila com os 3 da pilha\n");
        printf("0 - Sair\n");
        printf("Escolha: ");
        scanf("%d", &opcao);

        switch (opcao) {
            case 1: {
                if (!filaVazia(&fila)) {
                    removerFila(&fila);
                    inserirFila(&fila, gerarPeca());
                }
                break;
            }

            case 2: {
                if (!filaVazia(&fila) && !pilhaCheia(&pilha)) {
                    Peca p = removerFila(&fila);
                    push(&pilha, p);
                    inserirFila(&fila, gerarPeca());
                }
                break;
            }

            case 3: {
                if (!pilhaVazia(&pilha)) pop(&pilha);
                break;
            }

            case 4: {
                if (!filaVazia(&fila) && !pilhaVazia(&pilha)) {
                    Peca tempFila = fila.itens[fila.frente];
                    fila.itens[fila.frente] = pilha.itens[pilha.topo];
                    pilha.itens[pilha.topo] = tempFila;
                }
                break;
            }

            case 5:
                trocaTres(&fila, &pilha);
                break;

            case 0:
                printf("Encerrando...\n");
                break;
        }

    } while (opcao != 0);

    return 0;
}