#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define ORDEM 3 // Grau fixo da árvore B

typedef struct {
    char titulo[100];
    char autor[100];
    char genero[100];
    int ano;
    float preco;
    int codigo;
} Livro;

typedef struct BNo {
    int numChaves;
    Livro chaves[2 * ORDEM - 1];
    struct BNo* filhos[2 * ORDEM];
    int folha;
} BNo;

BNo* raiz = NULL;

// Função para criar um novo nó
BNo* criarNo(int folha) {
    BNo* novo = (BNo*)malloc(sizeof(BNo));
    novo->folha = folha;
    novo->numChaves = 0;
    for (int i = 0; i < 2 * ORDEM; i++) {
        novo->filhos[i] = NULL;
    }
    return novo;
}

// Função de divisão de filho
void dividirFilho(BNo* pai, int i, BNo* y) {
    BNo* z = criarNo(y->folha);
    z->numChaves = ORDEM - 1;

    for (int j = 0; j < ORDEM - 1; j++) {
        z->chaves[j] = y->chaves[j + ORDEM];
    }

    if (!y->folha) {
        for (int j = 0; j < ORDEM; j++) {
            z->filhos[j] = y->filhos[j + ORDEM];
        }
    }

    y->numChaves = ORDEM - 1;

    for (int j = pai->numChaves; j >= i + 1; j--) {
        pai->filhos[j + 1] = pai->filhos[j];
    }
    pai->filhos[i + 1] = z;

    for (int j = pai->numChaves - 1; j >= i; j--) {
        pai->chaves[j + 1] = pai->chaves[j];
    }

    pai->chaves[i] = y->chaves[ORDEM - 1];
    pai->numChaves++;
}

// Inserção não cheia
void inserirNaoCheio(BNo* no, Livro k) {
    int i = no->numChaves - 1;

    if (no->folha) {
        while (i >= 0 && strcmp(k.titulo, no->chaves[i].titulo) < 0) {
            no->chaves[i + 1] = no->chaves[i];
            i--;
        }
        no->chaves[i + 1] = k;
        no->numChaves++;
    } else {
        while (i >= 0 && strcmp(k.titulo, no->chaves[i].titulo) < 0) {
            i--;
        }
        i++;
        if (no->filhos[i]->numChaves == 2 * ORDEM - 1) {
            dividirFilho(no, i, no->filhos[i]);
            if (strcmp(k.titulo, no->chaves[i].titulo) > 0) {
                i++;
            }
        }
        inserirNaoCheio(no->filhos[i], k);
    }
}

// Inserção principal
void inserirLivro() {
    Livro novo;
    printf("Título: ");
    fgets(novo.titulo, sizeof(novo.titulo), stdin);
    novo.titulo[strcspn(novo.titulo, "\n")] = '\0';

    printf("Autor: ");
    fgets(novo.autor, sizeof(novo.autor), stdin);
    novo.autor[strcspn(novo.autor, "\n")] = '\0';

    printf("Gênero: ");
    fgets(novo.genero, sizeof(novo.genero), stdin);
    novo.genero[strcspn(novo.genero, "\n")] = '\0';

    printf("Ano de publicação: ");
    scanf("%d", &novo.ano);
    getchar();

    printf("Preço: ");
    scanf("%f", &novo.preco);
    getchar();

    novo.codigo = rand() % 90000 + 10000;

    if (raiz == NULL) {
        raiz = criarNo(1);
        raiz->chaves[0] = novo;
        raiz->numChaves = 1;
    } else {
        if (raiz->numChaves == 2 * ORDEM - 1) {
            BNo* novaRaiz = criarNo(0);
            novaRaiz->filhos[0] = raiz;
            dividirFilho(novaRaiz, 0, raiz);
            int i = 0;
            if (strcmp(novo.titulo, novaRaiz->chaves[0].titulo) > 0) {
                i++;
            }
            inserirNaoCheio(novaRaiz->filhos[i], novo);
            raiz = novaRaiz;
        } else {
            inserirNaoCheio(raiz, novo);
        }
    }

    printf("Livro adicionado com sucesso, código dele: %d\n", novo.codigo);
}

// Listagem ordenada
void mostrarLivros(BNo* no) {
    if (no != NULL) {
        int i;
        for (i = 0; i < no->numChaves; i++) {
            if (!no->folha) {
                mostrarLivros(no->filhos[i]);
            }
            printf("Livro: %s - Cod: %d\n", no->chaves[i].titulo, no->chaves[i].codigo);
        }
        if (!no->folha) {
            mostrarLivros(no->filhos[i]);
        }
    }
}

// Busca por título
void buscarLivro(BNo* no, const char* titulo) {
    if (no == NULL) {
        printf("Livro não encontrado.\n");
        return;
    }

    int i = 0;
    while (i < no->numChaves && strcmp(titulo, no->chaves[i].titulo) > 0) {
        i++;
    }

    if (i < no->numChaves && strcmp(titulo, no->chaves[i].titulo) == 0) {
        Livro l = no->chaves[i];
        printf("\nLivro encontrado:\nTítulo: %s\nAutor: %s\nGênero: %s\nAno: %d\nPreço: R$%.2f\nCódigo: %d\n",
               l.titulo, l.autor, l.genero, l.ano, l.preco, l.codigo);
        return;
    }

    if (no->folha) {
        printf("Livro não encontrado.\n");
    } else {
        buscarLivro(no->filhos[i], titulo);
    }
}

// Remoção por código (simples - apenas marca o livro como removido para fins de demonstração)
int removerLivro(BNo* no, int codigo) {
    if (no == NULL) return 0;

    for (int i = 0; i < no->numChaves; i++) {
        if (no->chaves[i].codigo == codigo) {
            printf("Livro removido: %s\n", no->chaves[i].titulo);
            for (int j = i; j < no->numChaves - 1; j++) {
                no->chaves[j] = no->chaves[j + 1];
            }
            no->numChaves--;
            return 1;
        }
    }

    if (!no->folha) {
        for (int i = 0; i <= no->numChaves; i++) {
            if (removerLivro(no->filhos[i], codigo)) return 1;
        }
    }

    return 0;
}

// Menu principal
void menu() {
    int opcao;
    char titulo[100];
    int codigo;

    do {
        printf("\n1 - Adicionar Livro\n2 - Mostrar Livros\n3 - Buscar Livro\n4 - Remover Livro\n5 - Sair\nOpção: ");
        scanf("%d", &opcao);
        getchar();

        if (opcao == 1) {
            inserirLivro();
        } else if (opcao == 2) {
            if (raiz == NULL || raiz->numChaves == 0) {
                printf("Sistema vazio. Nenhum livro registrado.\n");
            } else {
                mostrarLivros(raiz);
            }
        } else if (opcao == 3) {
            printf("Digite o título: ");
            fgets(titulo, sizeof(titulo), stdin);
            titulo[strcspn(titulo, "\n")] = '\0';
            buscarLivro(raiz, titulo);
        } else if (opcao == 4) {
            printf("Digite o código do livro: ");
            scanf("%d", &codigo);
            getchar();
            if (!removerLivro(raiz, codigo)) {
                printf("Código não localizado.\n");
            }
        } else if (opcao != 5) {
            printf("Opção inválida.\n");
        }
    } while (opcao != 5);
}

// Main
int main() {
    srand(time(NULL));
    menu();
    return 0;
}
