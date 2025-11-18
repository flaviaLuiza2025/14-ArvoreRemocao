# Árvores AVL - Remoção de Elementos
---

## Objetivos

Ao concluir esta atividade você deverá estender sua compreensão dos seguintes conceitos:
* Remoção de elementos em árvores binárias
* Manutenção das propriedades da árvore após remoção
* Tratamento dos diferentes casos de remoção

---

## Fundamentos Teóricos

### Como Funciona a Remoção em Árvores Binárias

A remoção de um elemento em uma árvore binária é mais complexa que a inserção, pois precisamos manter a estrutura e as propriedades da árvore após remover um nó. O processo pode ser dividido em **três casos principais**, dependendo da quantidade de filhos que o nó a ser removido possui.

#### **Caso 1: Nó Folha (Sem Filhos)**

Este é o caso mais simples. Quando o nó a ser removido não possui filhos (é uma folha):

1. Localize o nó a ser removido percorrendo a árvore
2. Identifique se ele é filho esquerdo ou direito de seu pai
3. Ajuste o ponteiro do pai para apontar para NULL (removendo a ligação)
4. Libere a memória do nó removido

**Exemplo visual:**
```
     10                    10
    /  \     remove 5     /  \
   5   15   --------->  NULL  15
```

#### **Caso 2: Nó com Um Filho**

Quando o nó a ser removido possui apenas um filho (esquerdo OU direito):

1. Localize o nó a ser removido
2. Identifique qual filho existe (esquerdo ou direito)
3. Conecte o pai do nó removido diretamente ao filho do nó removido
4. O filho "sobe" uma posição na árvore, ocupando o lugar do pai
5. Libere a memória do nó removido

**Exemplo visual:**
```
     10                    10
    /  \     remove 15    /  \
   5   15   --------->   5   20
        \
        20
```

#### **Caso 3: Nó com Dois Filhos**

Este é o caso mais complexo. Quando o nó possui ambos os filhos:

1. Localize o nó a ser removido
2. **Escolha o sucessor ou predecessor:**
   - **Sucessor:** o menor valor da subárvore direita (mais à esquerda)
   - **Predecessor:** o maior valor da subárvore esquerda (mais à direita)
3. Copie o valor do sucessor/predecessor para o nó a ser removido
4. Remova o sucessor/predecessor de sua posição original
   - Note que o sucessor/predecessor terá no máximo um filho (Casos 1 ou 2)
5. A remoção recursiva garante que a propriedade de busca seja mantida

**Exemplo visual usando sucessor:**
```
     10                    12
    /  \     remove 10    /  \
   5   15   --------->   5   15
      /  \                     \
     12  20                    20
     
O sucessor (12) substitui o nó removido (10)
```

### Por Que Usar o Sucessor ou Predecessor?

O sucessor (menor valor à direita) ou o predecessor (maior valor à esquerda) são escolhidos porque:
- Mantêm a propriedade de ordenação da árvore binária de busca
- O sucessor é maior que todos os valores da subárvore esquerda
- O predecessor é menor que todos os valores da subárvore direita
- Ambos têm no máximo um filho, simplificando sua remoção

### Considerações Importantes para Árvores AVL

Em árvores AVL, após qualquer remoção, é necessário:
1. Recalcular as alturas dos nós no caminho de volta
2. Verificar o fator de balanceamento de cada nó ancestral
3. Realizar rotações se necessário para manter o balanceamento
4. A árvore deve manter a propriedade AVL: |altura(esquerda) - altura(direita)| ≤ 1

---

## Como a Recursividade é Usada na Função `removerArvore`

A função `removerArvore` utiliza recursividade de forma elegante para realizar a remoção e manter as propriedades da árvore. Vamos entender como isso funciona:

### Estrutura Recursiva

```cpp
NO* removerArvore(NO* no, int valor) {
    // Caso base: nó não encontrado
    if (no == NULL) return no;
    
    // Recursão: navegar pela árvore
    if (valor < no->valor)
        no->esq = removerArvore(no->esq, valor);
    else if (valor > no->valor)
        no->dir = removerArvore(no->dir, valor);
    else {
        // Caso encontrado: realizar remoção
        // ...
    }
    
    // Backtracking: atualizar altura e balancear
    no->altura = maior(alturaNo(no->esq), alturaNo(no->dir)) + 1;
    return balancearNo(no);
}
```

### Fases da Recursividade

#### **1. Fase de Descida (Busca)**
Durante a descida recursiva, a função:
- Compara o valor procurado com o valor do nó atual
- Decide qual subárvore explorar (esquerda ou direita)
- Chama recursivamente a função na subárvore escolhida
- Cada chamada cria um novo contexto na pilha de execução

**Exemplo de pilha de chamadas:**
```
removerArvore(raiz=10, valor=5)
  └─> removerArvore(no=5, valor=5)  [nó encontrado!]
```

#### **2. Fase de Remoção**
Quando o nó é encontrado (`valor == no->valor`):
- Identifica qual dos 3 casos de remoção aplicar
- No Caso 3 (dois filhos), usa recursão novamente para remover o sucessor
- Esta é uma **recursão aninhada** dentro do próprio processo de remoção

**Recursão no Caso 3:**
```cpp
// Após copiar o valor do sucessor para o nó atual
no->dir = removerArvore(no->dir, sucessor->valor);
```

Isso significa: "remova o sucessor da subárvore direita, que será um caso mais simples (Caso 1 ou 2)"

#### **3. Fase de Subida (Backtracking)**
Após a remoção, durante o retorno das chamadas recursivas:
- Cada nó no caminho de volta é processado
- As alturas são recalculadas de baixo para cima
- O balanceamento é verificado e corrigido se necessário
- Isso garante que a propriedade AVL seja mantida em toda a árvore

**Fluxo de backtracking:**
```
removerArvore(no=5) → retorna novo ponteiro
  └─> atualiza no->esq em removerArvore(raiz=10)
    └─> atualiza altura e balanceia raiz
      └─> retorna nova raiz
```

### Por Que a Recursividade é Eficiente Aqui?

1. **Navegação Natural:** A estrutura recursiva da árvore se alinha perfeitamente com chamadas recursivas
2. **Código Conciso:** Evita loops complexos e gerenciamento manual de pilhas
3. **Balanceamento Automático:** O backtracking garante que todos os ancestrais sejam atualizados
4. **Tratamento Uniforme:** O Caso 3 reutiliza a própria função para remover o sucessor

### Visualizando a Recursão

```
Remover 10 de:
       10
      /  \
     5   15
        /  \
       12  20

1. Descida: removerArvore(10, 10) → nó encontrado
2. Caso 3: encontra sucessor (12)
3. Copia: 10 → 12
4. Recursão interna: removerArvore(15, 12)
5. Subida: atualiza altura de 15, depois de 12 (raiz)
6. Balanceia se necessário
7. Resultado:
       12
      /  \
     5   15
          \
          20
```

### Dicas para Implementação

- Sempre retorne o ponteiro do nó (pode ter mudado após rotações)
- Confie que a recursão tratará corretamente as subárvores
- O backtracking automático garante que todos os ancestrais sejam atualizados
- Teste cada caso isoladamente antes de integrar

---

## Atividade Proposta

Faça um fork deste repositório e realize as seguintes atividades: 

- [ ] **Implemente a função `removerElementoArvore`** que deve remover um elemento da árvore mantendo suas propriedades de busca e balanceamento
- [ ] **Trate corretamente os três casos de remoção:**
  - Nó folha (sem filhos)
  - Nó com um filho (esquerdo ou direito)
  - Nó com dois filhos (usando sucessor ou predecessor)
- [ ] **Teste sua implementação** com diferentes cenários:
  - Remover folhas
  - Remover nós intermediários com um filho
  - Remover nós com dois filhos
  - Remover a raiz da árvore
- [ ] **Verifique o balanceamento** da árvore após cada remoção (para árvores AVL)

### Dicas de Implementação

- Comece implementando a busca do nó a ser removido
- Identifique qual dos três casos se aplica
- Para o caso de dois filhos, implemente primeiro a busca do sucessor/predecessor
- Lembre-se de atualizar os ponteiros corretamente
- Não se esqueça de liberar a memória do nó removido
- Teste cada caso individualmente antes de integrar tudo

---

## Referências Adicionais

### Materiais de Estudo

- **Cormen, T. H. et al.** *Introduction to Algorithms*, 3rd Edition, MIT Press
  - Capítulo 12: Binary Search Trees (páginas 286-307)
  - Capítulo 13: Red-Black Trees (conceitos de balanceamento aplicáveis a AVL)

- **Sedgewick, R. & Wayne, K.** *Algorithms*, 4th Edition, Addison-Wesley
  - Seção 3.2: Binary Search Trees
  - Seção 3.3: Balanced Search Trees

### Vídeos e Tutoriais Online

- **Visualizador de Árvores AVL:** [VisuAlgo](https://visualgo.net/en/bst)
  - Ferramenta interativa para visualizar operações em árvores (inserção, remoção, rotações)
  
- **MIT OpenCourseWare:** [6.006 Introduction to Algorithms](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-006-introduction-to-algorithms-fall-2011/)
  - Lecture 6: AVL Trees, AVL Sort

- **GeeksforGeeks:** [AVL Tree | Set 2 (Deletion)](https://www.geeksforgeeks.org/avl-tree-set-2-deletion/)
  - Tutorial detalhado com exemplos de código

### Simuladores e Ferramentas

- **AVL Tree Visualization:** [University of San Francisco](https://www.cs.usfca.edu/~galles/visualization/AVLtree.html)
  - Simulador passo a passo de operações em árvores AVL
  
- **Binary Search Tree Visualizer:** [btv.melezinek.cz](https://www.btv.melezinek.cz/avl-tree.html)
  - Outra ferramenta de visualização com múltiplos exemplos

### Artigos Científicos e Documentação

- **Adelson-Velsky, G. & Landis, E. M.** (1962). "An algorithm for the organization of information"
  - Artigo original que introduziu as árvores AVL

- **Knuth, D. E.** *The Art of Computer Programming, Volume 3: Sorting and Searching*
  - Seção 6.2.3: Balanced Trees

### Exercícios Complementares

- **LeetCode:** Problemas relacionados
  - [Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/)
  - [Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/)

- **HackerRank:** [AVL Tree Collection](https://www.hackerrank.com/domains/data-structures/trees)
  - Problemas práticos de implementação


