---
title: understanding the bricktimize simplex algorithm
author: dirk olbrich
date: 2022-06-03
---

# Understanding the Bricktimize Simplex algorithm

These are some thoughts to understand the optimization algorithm to efficiently buy different pieces from Bricklink.com.

The basic ideas are taken from the [Bricklink helper][1] package by [Gianfranco Cecconi](https://github.com/giacecco).

The general math behind all this is to use the [Simplex algorithm][2], a linear programming algorithm to optimize linear equations with multiple constrains.

## The basic problem

I have a wishlist of Lego pieces which I want to buy from bricklink at the possible least overal price. Most sellers on Bricklink do not have all the pieces on my wishlist or they diverge on their price.

This problem can be described in generic terms:

I have a list of Lego pieces (items) **`I`** in different required quantities *`r`* each, which I want to buy. Each item *`i`* has a unique ID, which describes the item by type and color. This means that two items of the same type but with different color are separate items.

I adopt here the notion of *item* as this is the main reference in the Bricklink or Brickstore file format. An item can be a set, a part, a building instruction, a sticker set or any other piece of Lego swag. In terms of the optimization algorithm item referes to a Lego piece.

There are many sellers **`S`** on Bricklink, who can sell theses items. Each seller *`s`* has at least one item from my wanted items list. Each item is sold in condition *`c`* (`new` or `used`) at a specific price *`p`*.

I can set my preference for condition *`c`* to buy only `new` or `used` items. Or to set both condition types for the items I want to buy, prefering one. This means if i prefer `new` items but there are not enough on the market, the algorithm will fill the outstanding items with `used` ones.

Another thing to watch out for is that many seller have a minimum order value *`v`*. I have to check if the total price *`P`<sub>`s`</sub>* of all items I intend to order from seller `s` is above that threshold `v`, otherwise I can not order from this seller.

## Variables

- **`I`** is the total number of different items on my wanted list, where *`i`* defines the specific item.
    - If my list contains 3 different type of items, I want to buy **`I`** = 3 different items, regardless of the quantity I want to buy from each item.

- **`R`** is the total **required** number of all items **`I`** I want to complete my wanted list, where *`r`<sub>`i`</sub>* defines the required quantity of the single item.
    - If my list has 3 different items on it, with the quantity of each item defined with *`r`*<sub>`1`</sub> = 2, *`r`*<sub>`2`</sub> = 1, *`r`*<sub>`3`</sub> = 3, the total required quantity is **`R`** = 6.

- **`Q`** is the total **available** quantity of all items **`I`** from all seller **`S`**, where *`q`<sub>`i`</sub>* defines the available quantity of the single item *`i`*.

- **`S`** is the total number of sellers on Bricklink, from which I can buy items from my list, where *`s`* specifies the single seller. These seller offer at least 1 item on my list in at least a quantity of 1.

- **`P`** is the total price I would have to pay if I order all items on my wanted list. I want to optimize for **`P`** to be a Minimum.

- **`F`** is the fulfillment cost for each seller. This includes packaging, shipping and handling as well as placing the order and management of the order status. As it is not possible to obtain from Bricklink automatically the shipping cost for each seller, and as it may increase due to increased size of an order, I will use an appropriate general cost equal for all sellers.

From that we can define combined variables for *Items* , *Quantities* and *Sellers*, which describe the possible states of each.

- **`I`***<sub>`s`</sub>* is the total number of different items a seller *`s`* offers. A seller could offer all items on my list or just a subset of items.

- **`Q`***<sub>`s`</sub>* is the total available quantity of all items *`I`* combined from seller *`s`*.

- **`Q`***<sub>`i`</sub>* is the total available quantity of item *`i`* combined from all sellers *`S`*.

- *`q`<sub>`i,s`</sub>* is the available quantity of item *`i`* from seller *`s`*.

- *`r`<sub>`i`</sub>* is the required quantity of item *`i`*.

- **`O`***<sub>`s`</sub>* is the total ordered quantity of items from seller *`s`*.

- *`o`<sub>`i`,`s`</sub>* is the actual quantity of item *`i`* ordered from seller *`s`*.

- *`p`<sub>`i,s`</sub>* is the price for one item *`i`* from seller *`s`*.

- *`v`<sub>`s`</sub>* is the minimum total value of an order a seller *`s`* will accept, it can be zero or higher.

- *`c`* is the condition of the item `i`, either `N` = New or `U` = Used
    - thus the variable `o`<sub>i,s,c</sub> denotes the ordered quantity of item `i` from seller `s` in condition `c`
    - e.g. `o`<sub>`1,2,U`</sub> = 2 means I order 2x pieces of item 1 from seller 2 in a used condition

## Basic assumptions - constraints

1. A Seller should at least offer one single item, or else he is not a seller:

    *I<sub>s</sub>* &ge; 1 &and; *Q<sub>s</sub>* &ge; 1 &forall;*s*

    Latex:
    $$I_{s} \geq 1 \land Q_{s} \geq 1 \forall s$$

    For all seller `S` applies, that the total number of different available items from that seller must be greater or equal than 1 and that the total number of all available items must be greater or equal than 1.

2. I can't order from a seller more items than they got, nor do I want to order more items than I need:

    *o<sub>i,s</sub>* &le; min(*r<sub>i</sub>*, *q<sub>i,s</sub>*) &forall;*i,s*

    Latex:
    $$o_{i,s} \leq min(r_{i}, q_{i,s}) \forall i,s$$

    For all items `i` and all seller `s` applies, that the ordered quantity of item `i` from seller `s` is less than the required quantity of item `i` or the available quantity of item `i` from that seller `s`.

3. The total quantity of a specific item I order from all seller is not more than the required quantity of this item:

    &sum;<sup>*S*</sup><sub>*s*=1</sub> *o*<sub>*i*,*s*</sub> &le; *r*<sub>*i*</sub> &forall;*i*,*s*

    Latex:
    $$\sum^S_{s=1} o_{i,s} \leq r_{i} \forall i,s$$

    (&le; as there could be less available pieces of item *i* than required.)

## The algorithm

matrix constuction


1. total quantity of an item from all sellers is equal to the required quantity 

|               | o<sub>1,1</sub> | o<sub>2,1</sub> | o<sub>1,2</sub> | o<sub>2,2</sub> | ... | o<sub>i,s</sub> |     | b             |
| ------------- | --------------- | --------------- | --------------- | --------------- | --- | --------------- | --- | ------------- |
| i<sub>1</sub> | 1               | 0               | 1               | 0               | ... | 0               | =   | 5             |
| i<sub>2</sub> | 0               | 1               | 0               | 1               | ... | 0               | =   | 10            |
| ...           | ...             | ...             | ...             | ...             | ... | ...             |     | ...           |
| i<sub>n</sub> | 0               | 0               | 0               | 0               | ... | 1               | =   | r<sub>n</sub> |

## Example

Now on to a simple example to make all this math more tangible.

1. I want a set consisting of 2 different Lego pieces.

    |     | i1  | i2  |
    | --- | --- | --- |
    | r   | 5   | 10  |

2. Afters scraping Bricklink I got a list of 3 sellers offering these pieces in different quantities and at different prices. Each of these sellers has it's own minimum order value also.

    |     | i1  | i2  | p1   | p2   | v    |
    | --- | --- | --- | ---- | ---- | ---- |
    | s1  | 0   | 10  | 0    | 0.20 | 1.00 |
    | s2  | 10  | 20  | 0.50 | 0.30 | 2.00 |
    | s3  | 5   | 0   | 0.40 | 0    | 0    |

    Seller `s`<sub>`1`</sub> offers 10 pieces if item `i`<sub>`2`</sub> for a price of `p`<sub>`2`</sub> = 0.20 each. But seller `s`<sub>`1`</sub> has a minimum order value of `v` = 1.00 which I have to consider, meaning I have to order a quantity of at least `o`<sub>`i=2,s=1`</sub> = 5 pieces of item `i`<sub>`2`</sub> to reach this limit.

## References and Footnotes

[1]: https://github.com/giacecco/bricklink-helper
[2]: https://en.wikipedia.org/wiki/Simplex_algorithm
