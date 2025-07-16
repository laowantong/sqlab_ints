# Cheat sheet

## Exercices

### Exercice 1

**Token.** 002.

Un entier est un **carré parfait** si et seulement si sa racine carrée est entière.

| Exemple | Racine carrée | Propriété | Carré parfait |
|---:|:--:|:--:|:--:|
| $16$ | $4$ | entier | oui |
| $20$ | $$4,4721\dots$$ | non entier | non |

_Tâche._ Listez par ordre croissant les carrés parfaits inférieurs ou égaux à 1000.

**Formule**. `salt_002(sum(nn(hash)) OVER ()) AS token`

Solution recommandée. En ne gardant que les entiers dont la racine carrée a une partie fractionnaire nulle. Notez l'emploi inhabituel de l'opérateur modulo.

```sql
SELECT i
FROM ints
WHERE sqrt(i) % 1 = 0
```

Variante. En ne gardant que les entiers dont la racine carrée est différente de sa propre partie entière. Cela demande à calculer deux fois la racine carrée.

```sql
SELECT i
FROM ints
WHERE sqrt(i) = floor(sqrt(i))
```

Variante. En ne gardant que les entiers $a$ égaux au carré d'un entier $b$ (différent de $a$, sauf pour 0 et 1). Peu performant, du fait de l'auto-jointure.

```sql
SELECT A.i
FROM ints A
JOIN ints B ON B.i * B.i = A.i
```

Variante. Même idée, mais exprimée de façon plus procédurale, avec une requête imbriquée dans la clause `WHERE`. On a également borné $b$ à $\sqrt{1000} < 32$. Attention : si vous travaillez sur une table plus grande, vous devrez ajuster cette valeur.

```sql
SELECT i
FROM ints
WHERE i IN
        (SELECT i * i
         FROM ints
         WHERE i < 32 )
```

### Exercice 2

**Token.** 043.

Un entier est **triangulaire** si et seulement s'il peut s'écrire sous la forme $\frac{n (n+1)}{2}$ avec $n$ entier positif ou nul.

| Exemple | Forme cherchée | Triangulaire |
|---:|:--:|:--:|
| $15$ | $$5\times6\div2$$ | oui |
| $16$ | non | non |

_Tâche._ Listez par ordre croissant les nombres triangulaires inférieurs ou égaux à 1000.

**Formule** (remplacez (0) par le dixième nombre de la colonne). `salt_043((0) + sum(nn(A.hash)) OVER ()) AS token`

On parcourt tous les entiers $A_i$ de la liste, et on ne garde que ceux pour lesquels il existe un entier $B_i$ vérifiant l'équation. Comme on connaît la borne supérieure ($1000$, qui est plus petit $45\times 46\div 2=1035$), on peut (facultativement) insérer la « garde » `B.i < 45`.

```sql
SELECT i
FROM ints A
WHERE EXISTS
        (SELECT 1
         FROM ints B
         WHERE B.i < 45
             AND A.i = B.i * (B.i + 1) / 2 )
```

Variante. Avec une auto-jointure.

```sql
SELECT A.i
FROM ints A
JOIN ints B ON A.i = B.i * (B.i + 1) / 2
WHERE B.i < 45
ORDER BY A.i;
```

Variante. Avec une sous-requête dans le `WHERE`.

```sql
SELECT i
FROM ints
WHERE i IN
        (SELECT i * (i + 1) / 2
         FROM ints
         WHERE i < 45 )
```

### Exercice 3

**Token.** 023.

Un entier $n$ est **abondant** si et seulement s'il est inférieur à la somme de ses diviseurs stricts (_i.e._, distincts de $n$).

| Exemple | Diviseurs stricts | Propriété | Abondant |
|---:|:--:|:--:|:--:|
| $12$ | $${1, 2, 3, 4, 6}$$ | $$12 < 1+2+3+4+6 = 16$$ | oui |
| $16$ | $${1, 2, 4, 8}$$ | $$16 \geq 1+2+4+8 = 15$$ | non |

_Tâche._ Listez par ordre croissant les nombres abondants inférieurs ou égaux à 1000.

_Contrainte._ Utilisez une auto-jointure et un regroupement.

**Formule** (remplacez (0) par le 6e terme de cette suite). `salt_023((0) + bit_xor(sum(nn(A.hash) + nn(B.hash))) OVER ()) AS token`

```sql
SELECT A.i
FROM ints A
JOIN ints B ON B.i < A.i
AND A.i % B.i = 0
GROUP BY A.i
HAVING A.i < sum(B.i)
ORDER BY 1
```

### Exercice 4

**Token.** 024.

Un entier $n$ est **abondant** si et seulement s'il est inférieur à la somme de ses diviseurs stricts (_i.e._, distincts de $n$).

| Exemple | Diviseurs stricts | Propriété | Abondant |
|---:|:--:|:--:|:--:|
| $12$ | $${1, 2, 3, 4, 6}$$ | $$12 < 1+2+3+4+6 = 16$$ | oui |
| $16$ | $${1, 2, 4, 8}$$ | $$16 \geq 1+2+4+8 = 15$$ | non |

_Tâche._ Listez par ordre croissant les nombres abondants inférieurs ou égaux à 1000.

_Contrainte._ Utilisez une sous-requête corrélée, et pas de regroupement.

**Formule** (remplacez (0) par le 6e terme de cette suite). `salt_024((0) + sum(nn(A.hash)) OVER ()) AS token`

```sql
SELECT A.i
FROM ints A
WHERE A.i <
        (SELECT sum(B.i)
         FROM ints B
         WHERE B.i < A.i
             AND A.i % B.i = 0 )
ORDER BY 1
```

### Exercice 5

**Token.** 037.

Un entier est **sans facteur carré** si et seulement si aucun des nombres de sa décomposition en facteurs premiers n'apparaît plus d'une fois.

| Exemple | Décomposition | Propriété | Sans facteur carré |
|---:|:--:|:--:|:--:|
| $30$ | $$2 \times 3 \times 5$$ | aucun facteur dupliqué | oui |
| $12$ | $$2 \times 2 \times 3$$ | $2$ apparaît plus d'une fois | non |

_Tâche._ Listez par ordre croissant les entiers sans facteurs carrés inférieurs ou égaux à 1000.

**Formule**. `salt_037(bit_xor(sum(nn(A.hash) + nn(S.hash))) OVER ()) AS token`

```sql
WITH squares AS
    (SELECT DISTINCT B.i * B.i AS N2
                   , hash
     FROM ints B
     WHERE B.i * B.i <= 1000
         AND B.i > 1 )
SELECT A.i
FROM ints A
LEFT JOIN squares S ON A.i % S.n2 = 0
GROUP BY A.i
HAVING count(S.n2) = 0
```

### Exercice 6

**Token.** 032.

Un entier $c$ est **bicarré** si et seulement s'il peut s'écrire sous la forme $a^2+b^2$ avec $a$ et $b$ entiers.

| Exemple | Forme cherchée | Bicarré |
|---:|:--:|:--:|
| $17$ | $$1^2+4^2$$ | oui |
| $15$ | aucune | non |

<figure>
  <img src="https://www.dropbox.com/scl/fi/j9dtivl6qfcrs2nkv74xq/nombre-bigarr.png?rlkey=kb58znx2v7f69wsn1sy5hhw6q&raw=1"/>
  <figcaption>Entier bicarré bigarré.</figcaption>
</figure>

_Tâche._ Listez par ordre croissant les entiers bicarrés inférieurs ou égaux à 1000.

_Contrainte._ Faites un produit cartésien de trois tables.

**Formule** (remplacez (0) par le 6e terme de cette suite). `salt_032((0) + sum(nn(A.hash) + nn(B.hash) + nn(C.hash)) OVER ()) AS token`

```sql
SELECT DISTINCT C.i
FROM ints C
JOIN ints A ON A.i * A.i <= C.i
JOIN ints B ON A.i * A.i + B.i * B.i = C.i
ORDER BY 1
```

### Exercice 7

**Token.** 033.

Un entier $c$ est **bicarré** si et seulement s'il peut s'écrire sous la forme $a^2+b^2$ avec $a$ et $b$ entiers.

| Exemple | Forme cherchée | Bicarré |
|---:|:--:|:--:|
| $17$ | $$1^2+4^2$$ | oui |
| $15$ | aucune | non |

_Tâche._ Listez par ordre croissant les entiers bicarrés inférieurs ou égaux à 1000.

_Contrainte._ Faites un produit cartésien de deux tables seulement.

**Formule** (remplacez (0) par le 6e terme de cette suite). `salt_033((0) + sum(nn(A.hash) + nn(B.hash)) OVER ()) AS token`

On fait deux « boucles », l'une sur $a$, l'autre sur $b$. Plutôt que de « parcourir » tous les $c$ possibles, et éliminer ceux qui ne valent pas $a^2 + b^2$, on calcule directement $c = a^2 + b^2$, et on vérifie que cette somme est bien dans la table donnée (pour plus de généralité, on aurait pu écrire `A.i * A.i + B.i * B.i IN (SELECT i FROM ints)`). La deuxième condition du `ON` évite les doublons par symétrie (p. ex., (3, 4) et (4, 3)).

```sql
SELECT DISTINCT A.i * A.i + B.i * B.i AS i
FROM ints A
JOIN ints B ON A.i * A.i + B.i * B.i <= 1000
AND A.i <= B.i
ORDER BY 1
```

### Exercice 8

**Token.** 081.

Un entier est **premier** si et seulement s'il a exactement deux diviseurs entiers (1 et lui-même).

| Exemple | Diviseurs | Propriété | Premier |
|---:|:--:|:--:|:--:|
| $13$ | $${1, 13}$$ | exactement deux diviseurs | oui |
| $12$ | $${1, 2, 3, 4, 6, 12}$$ | plus de deux diviseurs | non |
| $1$ | $${1}$$ | moins de deux diviseurs | non |

_Tâche._ Listez par ordre croissant les nombres premiers inférieurs ou égaux à 1000.

_Contrainte._ N'utilisez pas de regroupement.

**Formule**. `salt_081(sum(nn(A.hash)) OVER ()) AS token`

```sql
SELECT i
FROM ints A
WHERE NOT EXISTS
        (SELECT 1
         FROM ints B
         WHERE B.i BETWEEN 2 AND A.i - 1
             AND A.i % B.i = 0 )
    AND i > 1
```

Variante. Avec une requête imbriquée dans le WHERE.

```sql
SELECT A.i
FROM ints A
WHERE A.i > 1
    AND A.i NOT IN
        (SELECT A2.i
         FROM ints A2
         JOIN ints B ON B.i BETWEEN 2 AND sqrt(A2.i)
         AND A2.i % B.i = 0)
```

### Exercice 9

**Token.** 082.

Un entier est **premier** si et seulement s'il a exactement deux diviseurs entiers (1 et lui-même).

| Exemple | Diviseurs | Propriété | Premier |
|---:|:--:|:--:|:--:|
| $13$ | $${1, 13}$$ | exactement deux diviseurs | oui |
| $12$ | $${1, 2, 3, 4, 6, 12}$$ | plus de deux diviseurs | non |
| $1$ | $${1}$$ | moins de deux diviseurs | non |

_Tâche._ Listez par ordre croissant les nombres premiers inférieurs ou égaux à 1000.

_Contrainte._ Utilisez un regroupement.

**Formule**. `salt_082(bit_xor(sum(nn(A.hash))) OVER ()) AS token`

```sql
SELECT A.i
FROM ints A
LEFT JOIN ints B ON B.i BETWEEN 2 AND sqrt(A.i)
AND A.i % B.i = 0
WHERE A.i > 1
GROUP BY A.i
HAVING count(B.i) = 0
```

