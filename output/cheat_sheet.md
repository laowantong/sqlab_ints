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

<figure>
  <img src="https://raw.githubusercontent.com/laowantong/sqlab_ints/refs/heads/main/assets/square-numbers.svg"/>
  <figcaption>Carrés parfaits.</figcaption>
</figure>

**Formule**. `salt_002(sum(nn(hash)) OVER ()) AS token`

Solution recommandée. En ne gardant que les entiers dont la racine carrée a une partie fractionnaire nulle. Notez l'emploi inhabituel de l'opérateur modulo.

```sql
SELECT n
FROM ints
WHERE sqrt(n) MOD 1 = 0
```

Variante. En ne gardant que les entiers dont la racine carrée est différente de sa propre partie entière. Cela demande à calculer deux fois la racine carrée.

```sql
SELECT n
FROM ints
WHERE sqrt(n) = floor(sqrt(n))
```

Variante. En ne gardant que les entiers $a$ égaux au carré d'un entier $b$ (différent de $a$, sauf pour 0 et 1). Peu performant, du fait de l'auto-jointure.

```sql
SELECT A.n
FROM ints A
JOIN ints B ON A.n = B.n * B.n
```

Variante. Même idée, mais exprimée de façon plus procédurale, avec une requête imbriquée dans la clause `WHERE`. On a également borné $b$ à $\sqrt{1000} < 32$. Attention : si vous travaillez sur une table plus grande, vous devrez ajuster cette valeur.

```sql
SELECT n
FROM ints
WHERE n IN
        (SELECT n * n
         FROM ints
         WHERE n < 32 )
```

Variante. Autre idée avec une auto-jointure, mais plus élégante : vérifier simplement que la racine carrée se trouve dans le tableau des entiers. Même problème potentiel de performance cependant.

```sql
SELECT n
FROM ints
WHERE sqrt(n) IN (
    SELECT n
    FROM ints
  )
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
SELECT n
FROM ints
WHERE cast(n AS CHAR) = reverse(cast(n AS CHAR))
```

Variante. MySQL est notoirement peu regardant sur les types. Dans la version ci-dessous, il convertit implicitement l’entier `n` en chaîne de caractères pour appliquer la fonction `REVERSE`, puis reconvertit le résultat en entier pour la comparaison. Cela dit, pour éviter toute ambiguïté ou comportement implicite, on préférera `CAST(n AS CHAR)`, plus rigoureux et plus portable.

```sql
SELECT n
FROM ints
WHERE n = reverse(n)
```

### Exercice 3

**Token.** 043.

Un entier est **triangulaire** si et seulement s'il peut s'écrire sous la forme $\frac{n (n+1)}{2}$ avec $n$ entier positif ou nul.

| Exemple | Forme cherchée | Triangulaire |
|---:|:--:|:--:|
| $15$ | $$5\times6\div2$$ | ✅ |
| $16$ | ❌ | ❌ |

_Tâche._ Listez par ordre croissant les nombres triangulaires inférieurs ou égaux à 1000.

<figure>
  <img src="https://raw.githubusercontent.com/laowantong/sqlab_ints/refs/heads/main/assets/triangular-numbers.svg"/>
  <figcaption>Nombres triangulaires.</figcaption>
</figure>

**Formule** (remplacez (0) par le dixième nombre de la colonne). `salt_043((0) + sum(nn(A.hash)) OVER ()) AS token`

On parcourt tous les entiers $A_i$ de la liste, et on ne garde que ceux pour lesquels il existe un entier $B_i$ vérifiant l'équation. Comme on connaît la borne supérieure ($1000$, qui est plus petit $45\times 46\div 2=1035$), on peut (facultativement) insérer la « garde » `B.n < 45`.

```sql
SELECT n
FROM ints A
WHERE EXISTS
        (SELECT 1
         FROM ints B
         WHERE B.n < 45
             AND A.n = B.n * (B.n + 1) / 2 )
```

Variante. Avec une auto-jointure.

```sql
SELECT A.n
FROM ints A
JOIN ints B ON A.n = B.n * (B.n + 1) / 2
WHERE B.n < 45
ORDER BY A.n;
```

Variante. Avec une sous-requête dans le `WHERE`.

```sql
SELECT n
FROM ints
WHERE n IN
        (SELECT n * (n + 1) / 2
         FROM ints
         WHERE n < 45 )
```

### Exercice 4

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
SELECT n
FROM ints
WHERE cast(n * n AS CHAR) LIKE concat('%', cast(n AS CHAR))
```

Variante. En extrayant le bon nombre de caractères à droite et en comparant. L'observation de l'exercice sur les palindromes reste valable ici : MySQL pourrait se passer des opérations de conversion explicite, ce qui rendrait certainement l'expression plus lisible.

```sql
SELECT n
FROM ints
WHERE right(cast(n * n AS CHAR), length(cast(n AS CHAR))) = cast(n AS CHAR)
```

### Exercice 5

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
SELECT DISTINCT C.n
FROM ints C
JOIN ints A ON A.n * A.n <= C.n
JOIN ints B ON A.n * A.n + B.n * B.n = C.n
ORDER BY 1
```

### Exercice 6

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

On fait deux « boucles », l'une sur $a$, l'autre sur $b$. Plutôt que de « parcourir » tous les $c$ possibles, et éliminer ceux qui ne valent pas $a^2 + b^2$, on calcule directement $c = a^2 + b^2$, et on vérifie que cette somme est bien dans la table donnée (pour plus de généralité, on aurait pu écrire `A.n * A.n + B.n * B.n IN (SELECT n FROM ints)`). La deuxième condition du `ON` évite les doublons par symétrie (p. ex., (3, 4) et (4, 3)).

```sql
SELECT DISTINCT A.n * A.n + B.n * B.n AS n
FROM ints A
JOIN ints B ON A.n * A.n + B.n * B.n <= 1000
AND A.n <= B.n
ORDER BY 1
```

### Exercice 7

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
SELECT n
FROM ints A
WHERE NOT EXISTS
        (SELECT 1
         FROM ints B
         WHERE B.n BETWEEN 2 AND sqrt(A.n)
             AND A.n MOD B.n = 0 )
    AND n > 1
```

Variante. Avec une requête imbriquée dans le WHERE.

```sql
SELECT A.n
FROM ints A
WHERE A.n > 1
    AND A.n NOT IN
        (SELECT A2.n
         FROM ints A2
         JOIN ints B ON B.n BETWEEN 2 AND sqrt(A2.n)
         AND A2.n MOD B.n = 0)
```

### Exercice 8

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
SELECT A.n
FROM ints A
LEFT JOIN ints B ON B.n BETWEEN 2 AND sqrt(A.n)
AND A.n MOD B.n = 0
WHERE A.n > 1
GROUP BY A.n
HAVING count(B.n) = 0
```

### Exercice 9

**Token.** 062.

Un entier est **composite** si et seulement s'il a plus de deux diviseurs entiers (1 et lui-même).

| n | Diviseurs |
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
SELECT A.n
     , group_concat(B.n
                    ORDER BY B.n ASC SEPARATOR ' ') AS divisors
FROM ints A
JOIN ints B ON A.n MOD B.n = 0
WHERE B.n = A.n
    OR B.n BETWEEN 1 AND sqrt(A.n)
GROUP BY A.n
HAVING count(B.n) > 2
```

### Exercice 10

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
SELECT A.n
FROM ints A
JOIN ints B ON B.n < A.n
AND A.n MOD B.n = 0
GROUP BY A.n
HAVING A.n < sum(B.n)
ORDER BY 1
```

### Exercice 11

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
SELECT A.n
FROM ints A
WHERE A.n <
        (SELECT sum(B.n)
         FROM ints B
         WHERE B.n < A.n
             AND A.n MOD B.n = 0 )
ORDER BY 1
```

### Exercice 12

**Token.** 037.

Un entier est **sans facteur carré** si et seulement si aucun des nombres de sa décomposition en facteurs premiers n'apparaît plus d'une fois.

| Exemple | Décomposition | Propriété | Sans facteur carré |
|---:|:--:|:--:|:--:|
| $30$ | $$2 \times 3 \times 5$$ | aucun facteur dupliqué | ✅ |
| $12$ | $$2 \times 2 \times 3$$ | $2$ apparaît plus d'une fois | ❌ |

_Tâche._ Listez par ordre croissant les entiers sans facteurs carrés inférieurs ou égaux à 1000.

**Formule**. `salt_037(sum(nn(hash)) OVER ()) AS token`

Pour chaque entier $i_a$, on vérifie qu'il n'existe aucun entier $i_s$ dont le carré divise $i_a$.

```sql
SELECT n
FROM ints A
WHERE n > 0
  AND NOT EXISTS (
    SELECT 1
    FROM ints AS S
    WHERE S.n >= 2
      AND S.n * S.n <= A.n
      AND A.n MOD (S.n * S.n) = 0
  )
```

Variante. Une CTE calcule la table `squares` des carrés susceptibles d'être facteurs d'entiers de `ints`. On met ensuite chaque ligne de `ints` en face de chacun de ses diviseurs carrés. La jointure externe permet de garder les lignes pour lesquelles aucune correspondance n'est possible : ce sont celles qui nous intéressent.

```sql
WITH
squares AS (
    SELECT DISTINCT n * n AS i2
    FROM ints
    WHERE n > 1 AND n * n <= 1000
)
SELECT n
FROM ints
LEFT JOIN squares ON n MOD i2 = 0
WHERE i2 is NULL
```

Variante. Même idée avec une sous-requête non corrélée. Mais il faut se méfier de la forme `NOT IN table` : si `table` contient ne serait-ce qu'un `NULL`, la condition renverra toujours `NULL` ! Pour vous en convaincre, testez avec `SELECT 3 NOT IN (1, 2, NULL), 3 NOT IN (1, 2)`. Ici, aucun `NULL` n'apparaît dans le résultat de la sous-requête, mais de façon générale évitez ce genre de requête au comportement potentiellement déroutant.

```sql
SELECT n
FROM ints
WHERE n NOT IN (
    SELECT A.n
    FROM ints AS A
    JOIN ints AS S ON A.n MOD (S.n * S.n) = 0
    WHERE S.n > 1 AND S.n * S.n <= 1000
)
```

### Exercice 13

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
(`n MOD 10 != 0`).

Pour chaque $A_i$, on vérifie si c'est un nombre de Kaprekar via la sous-requête `EXISTS` :

- On prend le même entier $A_i$ sous l'alias $B_i$.
- On parcourt toutes les positions de coupe jusque avant le dernier caractère de $B_i^2$.
- On découpe $B_i^2$ et on somme les parties.
- On s'assure que cette somme est égale à $A_i$.

```sql
SELECT n
FROM ints AS A
WHERE n MOD 10 != 0
    AND EXISTS
        (SELECT 1
         FROM ints AS cut
         JOIN ints AS B ON cut.n < length(B.n * B.n)
         WHERE A.n = B.n
             AND A.n = left(B.n * B.n, cut.n) + right(B.n * B.n, length(B.n * B.n) - cut.n) )
```

Variante. Avec une CTE qui précalcule les carrés une fois pour toutes.

```sql
WITH squares AS
    (SELECT n
          , n * n AS I2
     FROM ints)
SELECT n
FROM ints AS A
WHERE n MOD 10 != 0
    AND EXISTS
        (SELECT 1
         FROM ints AS cut
         JOIN squares ON cut.n < length(I2)
         WHERE A.n = squares.n
             AND A.n = left(I2, cut.n) + right(I2, length(I2) - cut.n) )
```

### Exercice 14

**Token.** 019.

La **suite de Fibonacci** commence par 0 et 1, ensuite chaque terme est la somme des deux précédents :

| Terme | Explication |
|-------:|:-------:|
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

_NB._ Cette suite comportant une répétion, vous ne pouvez pas l'exprimer comme une sous-séquence de la suite des entiers naturels. Ici, exceptionnellement, passez-vous de la table `ints`.

**Formule** (remplacez (0) par la concaténation de ces nombres.). `salt_019(string_hash('(0)') + count(*) OVER ()) AS token`

```sql
WITH RECURSIVE
fib (a, b) AS (
    SELECT 0, 1

    UNION ALL

    SELECT b, a + b
    FROM fib
    WHERE b <= 1000 )
SELECT a as n
FROM fib
```

### Exercice 15

**Token.** 039.

Un entier est un **nombre de Harshad** si et seulement s'il est divisible par la somme de ses chiffres (en écriture décimale).

| Exemple | Chiffres | Somme | Propriété | Harshad |
|---:|:--:|:--:|:--:|:--:|
| $42$ | $${4, 2}$$ | $6$ | divisible | ✅ |
| $11$ | $${1, 1}$$ | $2$ | non divisible | ❌ |

_Tâche._ Listez par ordre croissant les nombres de Harshad inférieurs ou égaux à 1000.

_Contrainte._ Utilisez une CTE récursive.

**Formule** (remplacez (0) par concaténation du nombre de lignes et des vingt premiers n). `salt_039(sum(string_hash('(0)')) OVER ()) AS token`

On construit récursivement une table avec tous les chiffres de chaque nombre. Ensuite, on regroupe par nombre. Enfin, on teste la propriété de divisibilité avec la somme des chiffres de chaque nombre.

```sql
WITH RECURSIVE
digits (n, q, r) AS
    (SELECT n
          , n MOD 10
          , n DIV 10
     FROM ints

     UNION ALL

     SELECT n
          , r MOD 10
          , r DIV 10
     FROM digits
     WHERE r > 0 )
SELECT n
FROM digits
GROUP BY n
HAVING n MOD sum(q) = 0
```

Variante. On peut calculer la somme au fur et à mesure de la récursion au lieu de reparcourir la table après coup. La table construite récursivement associe, à chaque nombre de $d$ chiffres, $d$ sommes intermédiaires. Attention : seule la dernière (`r = 0`) nous intéresse.

```sql
WITH RECURSIVE
digits(n, digit_sum, r) AS (
    SELECT n, n MOD 10, n DIV 10
    FROM ints
    
    UNION ALL
    
    SELECT n, digit_sum + (r MOD 10), r DIV 10
    FROM digits
    WHERE r > 0
)
SELECT n
FROM digits
WHERE r = 0 AND n MOD digit_sum = 0
```

### Exercice 16

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

**Formule** (remplacez (0) par la concaténation des 9e au 16e termes, séparés par des points, en minuscules.). `salt_088(string_hash('(0)') + sum(nn(hash)) OVER ()) AS token`

```sql
SELECT CASE
           WHEN n MOD 15 = 0 THEN 'FizzBuzz'
           WHEN n MOD 3 = 0 THEN 'Fizz'
           WHEN n MOD 5 = 0 THEN 'Buzz'
           ELSE n
       END AS fizzbuzz
FROM ints
WHERE n > 0
```

Variante. En générant dynamiquement la chaîne.

```sql
SELECT coalesce(nullif(concat(if(n MOD 3 = 0, 'Fizz', ''), if(n MOD 5 = 0, 'Buzz', '')), ''), n) AS fizzbuzz
FROM ints
WHERE n > 0
```

### Exercice 17

**Token.** 078.

Pas de bol : les vendredis 13 sont comparativement plus nombreux que les lundis 13, mardis 13, etc. Comme le notait déjà Arthur Schopenhauer ([Parerga und Paralipomena](https://www.youtube.com/watch?v=dQw4w9WgXcQ), Band II, Kapitel XXXI: _Zur Metaphysik des Aberglaubens_, § 394. Erste Ausgabe, A. W. Hayn, Berlin, 1851, S. 663) :

> « C'est une preuve supplémentaire de la méchanceté fondamentale de la Volonté, que le treizième jour d'un mois tombe plus souvent un vendredi que tout autre jour de la semaine. La Nature elle-même semble s'être liguée contre nous, nourrissant notre superstition avec une précision mathématique. Quelle cruelle ironie que même le calendrier devienne un instrument de notre souffrance ! »

_Tâche_. Vérifiez cette triste réalité en donnant, pour chaque jour de la semaine (à commencer par le lundi), le nombre de fois où il tombe le 13 du mois dans un cycle grégorien complet (400 ans). Attention : choisissez une année de départ postérieure au début du calendrier grégorien (1582).


**Formule** (remplacez (0) par la concaténation de ces nombres.). `salt_078(string_hash('(0)') + count(*) OVER ()) AS token`

```sql
WITH 
years AS (
    SELECT 1583 + n AS year_number
    FROM ints
    WHERE n < 400
),
days AS (
    SELECT n AS day_number
    FROM ints
    WHERE n BETWEEN 1 AND 366
),
dates AS (
    SELECT makedate(year_number, day_number) AS D
    FROM years, days
)
SELECT dayname(D) AS day
     , count(*) AS occurrences
FROM dates
WHERE day(D) = 13
GROUP BY weekday(D)
       , dayname(D)
ORDER BY weekday(D)
```

Variante. Même principe, mais en calculant les dates du 13 des 4800 mois qui commencent au 13 d'un mois arbitraire d'une année arbitraire.

```sql
WITH dates AS
    (SELECT date_add('1583-01-13', INTERVAL A.n + 1000 * B.n MONTH) AS D
     FROM ints AS A
     CROSS JOIN ints AS B
     WHERE A.n < 1000
         AND A.n + 1000 * B.n < 4800 )
SELECT dayname(D) AS `day`
     , count(D) AS `occurrences`
FROM dates
GROUP BY weekday(D)
       , dayname(D)
ORDER BY weekday(D)
```

### Exercice 18

**Token.** 047.

On se donne une table associant une sélection d'entiers à leur représentation en chiffres romains, dans cet ordre :

| Position | Arabe | Romain |
|---------:|------:|:------:|
|        1 |  1000 |  M     |
|        2 |   900 |  CM    |
|        3 |   500 |  D     |
|        4 |   400 |  CD    |
|        5 |   100 |  C     |
|        6 |    90 |  XC    |
|        7 |    50 |  L     |
|        8 |    40 |  XL    |
|        9 |    10 |  X     |
|       10 |     9 |  IX    |
|       11 |     5 |  V     |
|       12 |     4 |  IV    |
|       13 |     1 |  I     |

Pour convertir un entier $n$ quelconque, on applique l'algorithme suivant (code Python [ici](https://stackoverflow.com/a/47713392/173003)) :

1. Un accumulateur est initialisé à la chaîne vide.
2. Pour chaque ligne $(a, r)$ de la table d'association :
    - la division entière de $n$ par $a$ donne un quotient $q$ et un reste $m$ ;
    - $n$ devient $q$ ;
    - l'accumulateur est suffixé $m$ fois par $r$.
3. l'accumulateur contient la représentation désirée.

_Tâche._ Donnez, pour chaque entier de 1 à 1000, sa représentation en chiffres romains.

_Aide._ Vous aurez besoin de deux CTE :
1. La première construit la table d'association. Utilisez `VALUES`. La colonne `position` sert à rester en complexité linéaire lors de la conversion.
2. La seconde est récursive, et crée une table avec un certain nombre de colonnes (à déterminer) et dont certaines lignes (à déterminer) contiendront les résultats. Initialisez l'accumulateur à `CAST('' AS CHAR(20))` pour réserver l'espace nécessaire à sa croissance, et utilisez `CONCAT` et `REPEAT` pour le mettre à jour.

**Formule** (remplacez (0) par la concaténation des nombres romains en partant du 11e et par pas de 43). `salt_047(sum(string_hash('(0)')) OVER ()) AS token`

```sql
WITH RECURSIVE

map (pos, arabic, roman) AS (
    SELECT 1, 1000, 'M'
    UNION ALL SELECT 2, 900, 'CM'
    UNION ALL SELECT 3, 500, 'D'
    UNION ALL SELECT 4, 400, 'CD'
    UNION ALL SELECT 5, 100, 'C'
    UNION ALL SELECT 6, 90, 'XC'
    UNION ALL SELECT 7, 50, 'L'
    UNION ALL SELECT 8, 40, 'XL'
    UNION ALL SELECT 9, 10, 'X'
    UNION ALL SELECT 10, 9, 'IX'
    UNION ALL SELECT 11, 5, 'V'
    UNION ALL SELECT 12, 4, 'IV'
    UNION ALL SELECT 13, 1, 'I'
),

conversion (n, remaining, acc, pos) AS (
    SELECT n, n, CAST('' AS CHAR(20)), 1
    FROM ints
    WHERE n > 0
    
    UNION ALL
    
    SELECT 
        n,
        remaining MOD arabic,
        CONCAT(acc, REPEAT(roman, remaining DIV arabic)),
        pos + 1
    FROM conversion
    JOIN map USING (pos)
)

SELECT
    n,
    acc as roman
FROM conversion
WHERE pos = 14
```

Variante. Syntaxe plus moderne, mais moins portable.

```sql
WITH RECURSIVE

map (pos, arabic, roman) AS (
    VALUES 
    ROW( 1, 1000, 'M'),
    ROW( 2,  900, 'CM'),
    ROW( 3,  500, 'D'),
    ROW( 4,  400, 'CD'),
    ROW( 5,  100, 'C'),
    ROW( 6,   90, 'XC'),
    ROW( 7,   50, 'L'),
    ROW( 8,   40, 'XL'),
    ROW( 9,   10, 'X'),
    ROW(10,    9, 'IX'),
    ROW(11,    5, 'V'),
    ROW(12,    4, 'IV'),
    ROW(13,    1, 'I')
),

conversion (n, remaining, acc, pos) AS (
    SELECT n, n, CAST('' AS CHAR(20)), 1
    FROM ints
    WHERE n > 0
    
    UNION ALL
    
    SELECT 
        n,
        remaining MOD arabic,
        CONCAT(acc, REPEAT(roman, remaining DIV arabic)),
        pos + 1
    FROM conversion
    JOIN map USING (pos)
)

SELECT
    n,
    acc as roman
FROM conversion
WHERE pos = 14
```

