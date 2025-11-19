
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define ARQ_NOME "biblioteca.bin"
#define MAX_TITULO 100
#define MAX_AUTOR 100
#define MAX_ISBN 20

typedef struct {
    char titulo[MAX_TITULO];
    char autor[MAX_AUTOR];
    char isbn[MAX_ISBN];
    int ano;
    float preco;
} Livro;

void limpaBuffer(void);
long tamanho(FILE *fp);
void cadastrar(FILE *fp);
void consultar(FILE *fp);
void imprimir_livro(const Livro *l);


void limpaBuffer(void) {
    int c;
    while ((c = getchar()) != '\n' && c != EOF) { /* nada */ }
}

long tamanho(FILE *fp) {
    if (fp == NULL) return 0;
    if (fseek(fp, 0, SEEK_END) != 0) {
        perror("fseek (tamanho)");
        return 0;
    }
    long bytes = ftell(fp);
    if (bytes < 0) {
        perror("ftell (tamanho)");
        return 0;
    }
    return bytes / sizeof(Livro);
}

void cadastrar(FILE *fp) {
    if (fp == NULL) {
        printf("Arquivo nao aberto.\n");
        return;
    }

    Livro l;
    char buf[200];

    printf("Cadastro de novo livro\n");

    /* Títulos e strings: usar fgets e remover '\n' via strcspn */
    printf("Titulo: ");
    if (fgets(l.titulo, sizeof(l.titulo), stdin) == NULL) {
        printf("Erro na leitura do titulo.\n");
        return;
    }
    l.titulo[strcspn(l.titulo, "\n")] = '\0';

    printf("Autor: ");
    if (fgets(l.autor, sizeof(l.autor), stdin) == NULL) {
        printf("Erro na leitura do autor.\n");
        return;
    }
    l.autor[strcspn(l.autor, "\n")] = '\0';

    printf("ISBN: ");
    if (fgets(l.isbn, sizeof(l.isbn), stdin) == NULL) {
        printf("Erro na leitura do ISBN.\n");
        return;
    }
    l.isbn[strcspn(l.isbn, "\n")] = '\0';

    /* Ano (int) e Preco (float): ler com fgets e converter para evitar problemas de buffer */
    printf("Ano: ");
    if (fgets(buf, sizeof(buf), stdin) == NULL) {
        printf("Erro na leitura do ano.\n");
        return;
    }
    l.ano = atoi(buf);

    printf("Preco (ex: 39.90): ");
    if (fgets(buf, sizeof(buf), stdin) == NULL) {
        printf("Erro na leitura do preco.\n");
        return;
    }
    l.preco = (float) atof(buf);

    /* Gravar no final do arquivo */
    if (fseek(fp, 0, SEEK_END) != 0) {
        perror("fseek (cadastrar)");
        return;
    }
    size_t written = fwrite(&l, sizeof(Livro), 1, fp);
    if (written != 1) {
        perror("fwrite (cadastrar)");
        return;
    }
    /* garantir flush em disco */
    fflush(fp);

    printf("Livro cadastrado com sucesso!\n");
}

/* Consulta por índice/posição (0-based) e exibe o registro */
void consultar(FILE *fp) {
    if (fp == NULL) {
        printf("Arquivo nao aberto.\n");
        return;
    }

    long total = tamanho(fp);
    if (total == 0) {
        printf("Nenhum registro no arquivo.\n");
        return;
    }

    printf("Total de registros: %ld\n", total);
    printf("Informe o indice (0 a %ld) do registro a consultar: ", total - 1);

    int idx;
    if (scanf("%d", &idx) != 1) {
        printf("Entrada invalida.\n");
        limpaBuffer();
        return;
    }
    limpaBuffer();

    if (idx < 0 || idx >= total) {
        printf("Indice fora do intervalo.\n");
        return;
    }

    if (fseek(fp, idx * sizeof(Livro), SEEK_SET) != 0) {
        perror("fseek (consultar)");
        return;
    }

    Livro l;
    size_t rd = fread(&l, sizeof(Livro), 1, fp);
    if (rd != 1) {
        perror("fread (consultar)");
        return;
    }

    imprimir_livro(&l);
}

/* Função auxiliar para imprimir um livro */
void imprimir_livro(const Livro *l) {
    printf("---- Livro ----\n");
    printf("Titulo : %s\n", l->titulo);
    printf("Autor  : %s\n", l->autor);
    printf("ISBN   : %s\n", l->isbn);
    printf("Ano    : %d\n", l->ano);
    printf("Preco  : R$ %.2f\n", l->preco);
    printf("----------------\n");
}

/* --- main: abre/cria arquivo e menu simples --- */
int main(void) {
    FILE *fp = fopen(ARQ_NOME, "r+b"); /* tenta abrir existente */
    if (fp == NULL) {
        /* se não existir, cria com w+b */
        fp = fopen(ARQ_NOME, "w+b");
        if (fp == NULL) {
            perror("Erro ao abrir/criar arquivo");
            return 1;
        }
    }

    int opcao;
    do {
        printf("\n=== Biblioteca Pessoal ===\n");
        printf("1 - Cadastrar livro\n");
        printf("2 - Consultar livro por indice\n");
        printf("3 - Mostrar numero de registros\n");
        printf("0 - Sair\n");
        printf("Escolha: ");

        if (scanf("%d", &opcao) != 1) {
            printf("Entrada invalida. Tente novamente.\n");
            limpaBuffer();
            continue;
        }
        limpaBuffer();

        switch (opcao) {
            case 1:
                cadastrar(fp);
                break;
            case 2:
                consultar(fp);
                break;
            case 3:
                printf("Numero de registros: %ld\n", tamanho(fp));
                break;
            case 0:
                printf("Saindo...\n");
                break;
            default:
                printf("Opcao invalida.\n");
        }
    } while (opcao != 0);

    fclose(fp);
    return 0;
}
