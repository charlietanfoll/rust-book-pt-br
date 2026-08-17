# Entendendo a Posse (Ownership)

A posse é o recurso mais exclusivo do Rust e tem profundas implicações para o
resto da linguagem. Ela permite que o Rust garanta a segurança de memória sem a
necessidade de um coletor de lixo (garbage collector), por isso é importante
entender como a posse funciona. Neste capítulo, falaremos sobre a posse, bem
como sobre vários recursos relacionados: empréstimo (borrowing), fatias (slices)
e como o Rust organiza os dados na memória.