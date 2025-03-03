# [Garbage Collection](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html#t2)

_Process of looking ate heap memory, identifying unused objects (unreferenced objects), and freeing them._

**Referenced object** (in use):

* Some part of your program still maintains a pointer to that object;
* Referenciado por uma root.

**Root Reference**:

* Local variables: stored in the stack of the thread;
* Static variables: stored in the class type.

---

## Etapas

1. **Mark**: marcar objetos referenciados, através de uma flag no cabeçalho;
2. **Delete**: apagar objetos não marcados, ou seja, objetos não referenciados.
   * Approach 1:
     * Lista com espaços livres;
     * Problemas:
       * **fragmentação**;
       * alocação de memória é lenta;
   * Approach 2:
     * **Compactação**, logo não existe fragmentação;
     * Alocação de memória é rápida;
     * Problemas:
       * processo mais longo.

---

## Generational Garbage Collection

* Melhor performance;
* Normalmente, novos objetos têm menor tempo de vida, logo o GC é mais eficiente nas gerações mais novas.

### JVM Generations

* **Young**: onde os novos objetos estão alocados e envelhecem;
* **Old**: long survived objects;
* **Permanent**: metadata, classes e métodos.

#### Young Generation

<p align="center">
    <img src="./docs/lae-diagrams-GCYoungGeneration.svg" alt="Young Generation" align="center"/>
</p>

* Novos objetos são armazenados no Eden;
* Após o GC atuar, o Eden e um dos survivors fica vazio;
* Objetos sobreviventes alternam entre os survivors, até passarem para a Old Generation - promotion.

### Stop the World Events

* **Minor GC**: atua sobre a Young Generation;
* **Major GC**: atua sobre a Old Generation;
* **Full GC**: atua sobre toda a memória.

## Propriedades do GC

_G1 is a generational, incremental, parallel, mostly concurrent, stop-the-world, and evacuating garbage
collector_ ~ from [here](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-CE6F94B6-71AF-45D5-829E-DEADD9BA929D)

* **Generational**: Objetos são separados em gerações, baseadas no seu tempo de vida;
* **Incremental**: _An incremental garbage collector is any garbage-collector that can run incrementally (meaning that it can do a little work, then some more work, then some more work), instead of having to run the whole collection without interruption;_
* **Parallel**: _Multiple threads are used to speed up garbage collection;_
* **Mostly Concurrent**: _Managed threads can continue to run most of the time while the concurrent garbage collection thread is running;_
* **Stop-the-World**: _The execution of the program is suspended for GC till all objects in the heap are processed;_
* **Evacuating**: _Live objects found within selected memory areas to collect are copied into new memory areas, compacting them in the process. After an evacuation has been completed, the space previously occupied by live objects is reused for allocation by the application._

---
---

# [Finalization](https://www.oracle.com/technical-resources/articles/javase/finalization.html)

* Gestão dos recursos nativos associados com um objeto;
* Limpeza de objetos que o GC considerou inalcançáveis;
* Ciclo de vida de um objeto:
  * Created;
  * Unreachable;
  * Added to the finalization queue;
  * Finalized;
  * GC deleted.

**Problemas**:

* Delays the GC;
* Retenção de memória;
* Tempo de vida de objetor maior;
* Não existe garantia que a finalização será executada;
* Objeto pode ser ressuscitado.

**AVISO**: não usar as seguintes funções:

* `System.gc()`: executa o GC;
* `System.runFinalization()`: executa os métodos de finalização;

---

## [try-with-resources statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)

* try statement que declara um ou mais recursos;
* Um recurso é um objeto que deve ser closed após a sua utilização.

Os recursos podem ser:

* Declarados no bloco try, e são fechados automaticamente após o seu scope;
  * Apenas em Java;
  * Exemplo: `try (FileWriter fw = new FileWriter("file.txt")) {...}`;
* Declarados antes do bloco try, e são fechados manualmente no block finally;
  * Exemplo: `try { FileWriter fw = new FileWriter("file.txt") } finally { fw.close() }`;
* Método `.use(block: (T) -> R): T`;
  * Apenas em Kotlin.

**NOTA**: **Closeable** e **AutoCloseable** são interfaces equivalentes; o AutoCloseable foi introduzido com o try-with-resources statement em Java, logo em Kotlin são equivalentes.

Uma classes com um campo Closeable/AutoCloseable, também deve implementar essa interface.

---
---

# [Cleaner](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/ref/Cleaner.html)

* Manages a set of object references and corresponding cleaning actions;
* Invocação explícita do método `clean()` quando o objeto é closed ou não é necessário;
* Interface `Cleanable` representa um objeto e uma cleaning action registados num Cleaner;
  * Contém um único método `clean()`, que executa a cleaning action e remove o registo do objeto.

`Cleanable register(Object obj, Runnable action)`: regista um objeto e uma cleaning action num Cleaner.