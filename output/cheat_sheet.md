# Cheat sheet

## Exercices

### Exercice 1

**Token.** 002.

Un entier est un **carré parfait** si et seulement si sa racine carrée est entière.

| Exemple | Racine carrée | Propriété | Carré parfait |
|---:|:--:|:--:|:--:|
| $16$ | $4$ | entière | ✅ |
| $20$ | $$4,4721\dots$$ | non entière | ❌ |

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
JOIN ints B ON A.i = B.i * B.i
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

**Token.** 052.

Un entier est **palindromique** si sa représentation décimale se lit de la même façon de gauche à droite et de droite à gauche.

| Exemple | De droite à gauche | Palindromique |
|---:|:--:|:--:|
| $7$ | $7$ | ✅ |
| $121$ | $121$ | ✅ |
| $123$ | $$321$$ | ❌ |

_Tâche._ Listez par ordre croissant les entiers palindromiques inférieurs ou égaux à 1000.

**Formule**. `salt_052(sum(nn(hash)) OVER ()) AS token`

Il suffit de comparer la chaîne correspondante à son inverse.

```sql
SELECT i
FROM ints
WHERE cast(i AS CHAR) = reverse(cast(i AS CHAR))
```

Variante. MySQL est notoirement peu regardant sur les types. Dans la version ci-dessous, il convertit implicitement l’entier `i` en chaîne de caractères pour appliquer la fonction `REVERSE`, puis reconvertit le résultat en entier pour la comparaison. Cela dit, pour éviter toute ambiguïté ou comportement implicite, on préférera `CAST(i AS CHAR)`, plus rigoureux et plus portable.

```sql
SELECT i
FROM ints
WHERE i = reverse(i)
```

### Exercice 3

**Token.** 043.

Un entier est **triangulaire** si et seulement s'il peut s'écrire sous la forme $\frac{n (n+1)}{2}$ avec $n$ entier positif ou nul.

| Exemple | Forme cherchée | Triangulaire |
|---:|:--:|:--:|
| $15$ | $$5\times6\div2$$ | ✅ |
| $16$ | ❌ | ❌ |

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

### Exercice 4

**Token.** 088.

La suite **FizzBuzz** est une transformation des entiers naturels strictement positifs où :

- tout multiple de 3 est remplacé par `'Fizz'` ;
- tout multiple de 5 est remplacé par `'Buzz'` ;
- tout multiple de 15 est remplacé par `'FizzBuzz'` ;
- les autres nombres restent inchangés.

| fizzbuzz     |
|:------------:|
| `1`          |
| `2`          |
| `Fizz`     |
| `4`          |
| `Buzz`     |
| `Fizz`     |
| `7`          |
| ⋮            |
| `14`         
| `FizzBuzz` |
| `16`         
| ⋮            |


_Tâche._ Listez par ordre croissant les termes de FizzBuzz jusqu'à 1000.

_Aide._ L'opérateur `CASE` fera ici merveille sous la forme suivante :

```sql
CASE
    WHEN condition_1 THEN result_1
    WHEN condition_2 THEN result_2
    ...
    ELSE default_result
END
```

**Formule** (remplacez (0) par la concaténation des 9e au 16e terme, séparés par des points, en minuscules.). `salt_088(string_hash('(0)') + sum(nn(hash)) OVER ()) AS token`

```sql
SELECT CASE
           WHEN i % 15 = 0 THEN 'FizzBuzz'
           WHEN i % 3 = 0 THEN 'Fizz'
           WHEN i % 5 = 0 THEN 'Buzz'
           ELSE i
       END AS fizzbuzz
FROM ints
WHERE i > 0
```

Variante. En générant dynamiquement la chaîne.

```sql
SELECT coalesce(nullif(concat(if(i % 3 = 0, 'Fizz', ''), if(i % 5 = 0, 'Buzz', '')), ''), i) AS fizzbuzz
FROM ints
WHERE i > 0
```

### Exercice 5

**Token.** 010.

Un entier $n$ est **automorphe** si son carré se termine par $n$ (en écriture décimale).

| Exemple | Carré | Automorphe |
|---:|:--:|:--:|
| $5$ | $25$ | ✅
| $25$ | $625$ | ✅ |
| $7$ | $49$ | ❌ |

_Tâche._ Listez par ordre croissant les entiers automorphes inférieurs ou égaux à 1000.

**Formule**. `salt_010(sum(nn(hash)) OVER ()) AS token`

Avec la fonction de concaténation et l'opérateur `LIKE`.

```sql
SELECT i
FROM ints
WHERE cast(i * i AS CHAR) LIKE concat('%', cast(i AS CHAR))
```

Variante. En extrayant le bon nombre de caractères à droite et en comparant. L'observation de l'exercice sur les palindromes reste valable ici : MySQL pourrait se passer des opérations de conversion explicite, ce qui rendrait certainement l'expression plus lisible.

```sql
SELECT i
FROM ints
WHERE right(cast(i * i AS CHAR), length(cast(i AS CHAR))) = cast(i AS CHAR)
```

### Exercice 6

**Token.** 032.

Un entier $c$ est **bicarré** si et seulement s'il peut s'écrire sous la forme $a^2+b^2$ avec $a$ et $b$ entiers.

| Exemple | Forme cherchée | Bicarré |
|---:|:--:|:--:|
| $17$ | $$1^2+4^2$$ | ✅ |
| $15$ | aucune | ❌ |

_Tâche._ Listez par ordre croissant les entiers bicarrés inférieurs ou égaux à 1000.

_Contrainte._ Faites un produit cartésien de trois tables.

<figure>
  <img src="https://raw.githubusercontent.com/laowantong/sqlab_ints/refs/heads/main/assets/nombre-bigarré.png"/>
  <figcaption>Entier bicarré et bigarré.</figcaption>
</figure>

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
| $17$ | $$1^2+4^2$$ | ✅ |
| $15$ | aucune | ❌ |

_Tâche._ Listez par ordre croissant les entiers bicarrés inférieurs ou égaux à 1000.

_Contrainte._ Faites un produit cartésien de deux tables seulement.

<figure>
  <img src="https://raw.githubusercontent.com/laowantong/sqlab_ints/refs/heads/main/assets/nombre-non-bigarré.png"/>
  <figcaption>Entier non bicarré ni bigarré.</figcaption>
</figure>

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
| $13$ | $${1, 13}$$ | exactement deux diviseurs | ✅ |
| $12$ | $${1, 2, 3, 4, 6, 12}$$ | plus de deux diviseurs | ❌ |
| $1$ | $${1}$$ | moins de deux diviseurs | ❌ |

_Tâche._ Listez par ordre croissant les nombres premiers inférieurs ou égaux à 1000.

_Contrainte._ N'utilisez pas de regroupement.

**Formule**. `salt_081(sum(nn(A.hash)) OVER ()) AS token`

```sql
SELECT i
FROM ints A
WHERE NOT EXISTS
        (SELECT 1
         FROM ints B
         WHERE B.i BETWEEN 2 AND sqrt(A.i)
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
| $13$ | $${1, 13}$$ | exactement deux diviseurs | ✅ |
| $12$ | $${1, 2, 3, 4, 6, 12}$$ | plus de deux diviseurs | ❌ |
| $1$ | $${1}$$ | moins de deux diviseurs | ❌ |

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

### Exercice 10

**Token.** 062.

Un entier est **composite** si et seulement s'il a plus de deux diviseurs entiers (1 et lui-même).

| i | Diviseurs |
|---:|:--|
|4 | 1 2 4 |
|6 | 1 2 6 |
|8 | 1 2 8 |
|9 | 1 3 9 |
|10 | 1 2 10 |
|12 | 1 2 3 12 |

_Tâche._ Listez par ordre croissant les nombres composites inférieurs ou égaux à 1000 avec la liste de leurs diviseurs séparés un espace comme dans la table ci-dessus.

_Aide._ Une simple variation du calcul des nombres premiers pour découvrir la fonction d'agrégation [`group_concat()`](https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html#function_group-concat).

**Formule** (remplacez (0) par la 14e liste de diviseurs, séparés par un espace). `salt_062(string_hash('(0)') + bit_xor(sum(nn(A.hash))) OVER ()) AS token`

```sql
SELECT A.i
     , group_concat(B.i
                    ORDER BY B.i ASC SEPARATOR ' ') AS divisors
FROM ints A
JOIN ints B ON A.i % B.i = 0
WHERE B.i = A.i
    OR B.i BETWEEN 1 AND sqrt(A.i)
GROUP BY A.i
HAVING count(B.i) > 2
```

### Exercice 11

**Token.** 023.

Un entier $n$ est **abondant** si et seulement s'il est inférieur à la somme de ses diviseurs stricts (_i.e._, distincts de $n$).

| Exemple | Diviseurs stricts | Propriété | Abondant |
|---:|:--:|:--:|:--:|
| $12$ | $${1, 2, 3, 4, 6}$$ | $$12 < 1+2+3+4+6 = 16$$ | ✅ |
| $16$ | $${1, 2, 4, 8}$$ | $$16 \geq 1+2+4+8 = 15$$ | ❌ |

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

### Exercice 12

**Token.** 024.

Un entier $n$ est **abondant** si et seulement s'il est inférieur à la somme de ses diviseurs stricts (_i.e._, distincts de $n$).

| Exemple | Diviseurs stricts | Propriété | Abondant |
|---:|:--:|:--:|:--:|
| $12$ | $${1, 2, 3, 4, 6}$$ | $$12 < 1+2+3+4+6 = 16$$ | ✅ |
| $16$ | $${1, 2, 4, 8}$$ | $$16 \geq 1+2+4+8 = 15$$ | ❌ |

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

### Exercice 13

**Token.** 037.

Un entier est **sans facteur carré** si et seulement si aucun des nombres de sa décomposition en facteurs premiers n'apparaît plus d'une fois.

| Exemple | Décomposition | Propriété | Sans facteur carré |
|---:|:--:|:--:|:--:|
| $30$ | $$2 \times 3 \times 5$$ | aucun facteur dupliqué | ✅ |
| $12$ | $$2 \times 2 \times 3$$ | $2$ apparaît plus d'une fois | ❌ |

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

### Exercice 14

**Token.** 009.

Un entier $n$ est un **nombre de Kaprekar** si et seulement si son carré peut être séparé en une partie gauche et une partie droite dont la somme vaut $n$. La partie gauche peut être vide. La partie droite ne peut être vide ou nulle.

| Exemple | Carré | Découpage       | Somme | Kaprekar                 |
|--------:|------:|----------------:|------:|:-------------------------|
| 1       | 1     | `"" + "1"`      | 1     | ✅ (NB : partie gauche vide)  |
| 5       | 25    | `"" + "25"` <br> `"2" + "5"`    | 25 <br> 7 | ❌                       |
| 9       | 81    | `"8" + "1"`     | 9     | ✅                       |
| 45      | 2025  | `"20" + "25"`   | 45    | ✅                       |
| 10      | 100   | `"10" + "0"`    | 10    | ❌ (partie droite nulle) |
| 99      | 9801  | `"98" + "01"`   | 99    | ✅                       |

_Tâche._ Listez par ordre croissant les nombres de Kaprekar inférieurs ou égaux à 1000.

_Aide._ Utilisez les fonctions `left(str, len)` et `right(str, len)`.

**Formule**. `salt_009(sum(nn(hash)) OVER ()) AS token`


On parcourt tous les entiers $A_i$ candidats, en excluant immédiatement les multiples de $10$
(`i % 10 != 0`).

Pour chaque $A_i$, on vérifie si c'est un nombre de Kaprekar via la sous-requête `EXISTS` :

- On prend le même entier $A_i$ sous l'alias $B_i$.
- On parcourt toutes les positions de coupe jusque avant le dernier caractère de $B_i^2$.
- On découpe $B_i^2$ et on somme les parties.
- On s'assure que cette somme est égale à $A_i$.

```sql
SELECT i
FROM ints AS A
WHERE i % 10 != 0
    AND EXISTS
        (SELECT 1
         FROM ints AS cut
         JOIN ints AS B ON cut.i < length(B.i * B.i)
         WHERE A.i = B.i
             AND A.i = left(B.i * B.i, cut.i) + right(B.i * B.i, length(B.i * B.i) - cut.i) )
```

Variante. Avec une CTE qui précalcule les carrés une fois pour toutes.

```sql
WITH squares AS
    (SELECT i
          , i * i AS I2
     FROM ints)
SELECT i
FROM ints AS A
WHERE i % 10 != 0
    AND EXISTS
        (SELECT 1
         FROM ints AS cut
         JOIN squares ON cut.i < length(I2)
         WHERE A.i = squares.i
             AND A.i = left(I2, cut.i) + right(I2, length(I2) - cut.i) )
```

### Exercice 15

**Token.** 019.

La **suite de Fibonacci** commence par 0 et 1, puis chaque autre terme est la somme des deux termes précédents :

| Terme | Explication |
|-------:|:-------|
| $0$      | (donné) |
| $1$      | (donné) |
| $1$      | $$0 + 1 = 1$$ |
| $2$      | $$1 + 1 = 2$$ |
| $3$      | $$1 + 2 = 3$$ |
| $5$      | $$2 + 3 = 5$$ |
| $8$      | $$3 + 5 = 8$$ |
| $13$     | $$5 + 8 = 13$$ |
|  ⋮  |   |

_Tâche._ Listez par ordre croissant tous les nombres de Fibonacci inférieurs ou égaux à 1000.

_Aide._ Utilisez une CTE (_Common Table Expression_) récursive.

_Contrainte._ Pour que SQLab puisse valider votre requête, vous devrez impérativement l'insérer dans la structure de filtrage suivante :

```sql
WITH RECURSIVE fib (
        /* en-tête de votre CTE */
    )
    AS (
        /* corps de votre CTE */
    )
SELECT i
FROM ints
WHERE i IN (SELECT i from fib)
```

**Formule**. `salt_019(sum(nn(hash)) OVER ()) AS token`

```sql
WITH RECURSIVE fib(i, J) AS
    (SELECT 1 AS i
          , 1 AS J

     UNION ALL

     SELECT J AS i
                    , i + J AS J
     FROM fib
     WHERE J <= 1000 )
SELECT i
FROM ints
WHERE i IN
        (SELECT i
         FROM fib)
```

### Exercice 16

**Token.** 039.

Un entier est un **nombre de Harshad** si et seulement s'il est divisible par la somme de ses chiffres (en écriture décimale).

| Exemple | Chiffres | Somme | Propriété | Harshad |
|---:|:--:|:--:|:--:|:--:|
| $42$ | $${4, 2}$$ | $6$ | divisible | ✅ |
| $11$ | $${1, 1}$$ | $2$ | non divisible | ❌ |

_Tâche._ Listez par ordre croissant les nombres de Harshad inférieurs ou égaux à 1000.

_Contrainte._ Utilisez une CTE récursive.

**Formule**. `salt_039(bit_xor(sum(nn(hash))) OVER ()) AS token`

```sql
WITH RECURSIVE digits(i, Q, R, hash) AS
    (SELECT i
          , i MOD 10
                , i DIV 10
                , hash
     FROM ints

     UNION ALL

     SELECT i
                    , R MOD 10
                          , R DIV 10
                          , hash
     FROM digits
     WHERE R > 0 )
SELECT i
--   , group_concat(q SEPARATOR ' ') as digits
FROM digits
GROUP BY i
HAVING i % sum(q) = 0
```

