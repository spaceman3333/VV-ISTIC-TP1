# Practical Session #1: Introduction

1. Find in news sources a general public article reporting the discovery of a software bug. Describe the bug. If possible, say whether the bug is local or global and describe the failure that manifested its presence. Explain the repercussions of the bug for clients/consumers and the company or entity behind the faulty program. Speculate whether, in your opinion, testing the right scenario would have helped to discover the fault.

2. Apache Commons projects are known for the quality of their code and development practices. They use dedicated issue tracking systems to discuss and follow the evolution of bugs and new features. The following link https://issues.apache.org/jira/projects/COLLECTIONS/issues/COLLECTIONS-794?filter=doneissues points to the issues considered as solved for the Apache Commons Collections project. Among those issues find one that corresponds to a bug that has been solved. Classify the bug as local or global. Explain the bug and the solution. Did the contributors of the project add new tests to ensure that the bug is detected if it reappears in the future?

3. Netflix is famous, among other things we love, for the popularization of *Chaos Engineering*, a fault-tolerance verification technique. The company has implemented protocols to test their entire system in production by simulating faults such as a server shutdown. During these experiments they evaluate the system's capabilities of delivering content under different conditions. The technique was described in [a paper](https://arxiv.org/ftp/arxiv/papers/1702/1702.05843.pdf) published in 2016. Read the paper and briefly explain what are the concrete experiments they perform, what are the requirements for these experiments, what are the variables they observe and what are the main results they obtained. Is Netflix the only company performing these experiments? Speculate how these experiments could be carried in other organizations in terms of the kind of experiment that could be performed and the system variables to observe during the experiments.

4. [WebAssembly](https://webassembly.org/) has become the fourth official language supported by web browsers. The language was born from a joint effort of the major players in the Web. Its creators presented their design decisions and the formal specification in [a scientific paper](https://people.mpi-sws.org/~rossberg/papers/Haas,%20Rossberg,%20Schuff,%20Titzer,%20Gohman,%20Wagner,%20Zakai,%20Bastien,%20Holman%20-%20Bringing%20the%20Web%20up%20to%20Speed%20with%20WebAssembly.pdf) published in 2018. The goal of the language is to be a low level, safe and portable compilation target for the Web and other embedding environments. The authors say that it is the first industrial strength language designed with formal semantics from the start. This evidences the feasibility of constructive approaches in this area. Read the paper and explain what are the main advantages of having a formal specification for WebAssembly. In your opinion, does this mean that WebAssembly implementations should not be tested? 

5.  Shortly after the appearance of WebAssembly another paper proposed a mechanized specification of the language using Isabelle. The paper can be consulted here: https://www.cl.cam.ac.uk/~caw77/papers/mechanising-and-verifying-the-webassembly-specification.pdf. This mechanized specification complements the first formalization attempt from the paper. According to the author of this second paper, what are the main advantages of the mechanized specification? Did it help improving the original formal specification of the language? What other artifacts were derived from this mechanized specification? How did the author verify the specification? Does this new specification removes the need for testing?

## Answers

## 1. https://www.msn.com/en-us/news/technology/google-uses-ai-to-discover-20-year-old-software-bug/ar-AA1urWKO

### Décrivez le bug

Le bug est une vulnérabilité dans la bibliothèque **OpenSSL** qui provoque un out-of-bound memory access. Ce type de problème peut entraîner des crashs de programme ou, dans de rares cas, permettre à un attaquant d’exécuter du code malveillant.

---
### Le bug est-il local ou global ?

Le bug est **global** car la bibliothèque OpenSSL est utilisée par de nombreuses applications et services Internet pour des communications sécurisées. Sa portée dépasse largement une utilisation spécifique ou locale.

---
### Décrivez la défaillance ayant révélé le bug

La faille s'est manifestée par des **crashs inattendus des programmes**. Elle avait également le potentiel, bien que limité, de permettre l'exécution de code malveillant à distance par un attaquant exploitant cette vulnérabilité.

---
### Expliquez les répercussions du bug

#### Pour les clients et consommateurs :
1. **Risques de sécurité** : La vulnérabilité pouvait être exploitée pour exécuter du code malveillant, mettant les données sensibles en danger.
2. **Interruption de service** : Les exceptions d’accès hors limites à la mémoire pouvaient provoquer des interruptions de service et affecter les performances des applications.

#### Pour la société ou entité responsable :
1. **Impact sur la réputation** : Si des utilisateurs malveillants avaient découvert et exploité ce bug avant sa correction, cela aurait pu gravement nuire à la confiance des clients envers la société.
2. **Coûts opérationnels** : Identifier, corriger et déployer des correctifs pour ce type de vulnérabilité nécessite des ressources importantes.

---
### Selon vous, un scénario de test approprié aurait-il permis de découvrir cette faille ?

Non, à mon avis, cette faille n’aurait pas été découverte par des tests traditionnels. Les chercheurs ont utilisé la méthode de **fuzz testing**, une technique basée sur des données aléatoires, pour identifier cette vulnérabilité. Ils ont affirmé que les cibles de tests écrites par des humains n’auraient pas permis de détecter cette faille ancienne.


## 2. [UnmodifiableNavigableSet can be modified by pollFirst() and pollLast()](https://issues.apache.org/jira/projects/COLLECTIONS/issues/COLLECTIONS-799?filter=doneissues) bug

This is a design bug since ```UnmodifiableNavigableSet``` must be unmodifiable according to its specification. The issue arises from the absence of overridden methods for ```pollFirst()``` and ```pollLast()``` in the class. 

This is a local bug.

The [solution](https://github.com/apache/commons-collections/pull/250) is simple and involves simply adding the following lines of code:

```java
    /**
     * @since 4.5
     */
    @Override
    public E pollFirst() {
        throw new UnsupportedOperationException();
    }

    /**
     * @since 4.5
     */
    @Override
    public E pollLast() {
        throw new UnsupportedOperationException();
    }
```

The contributor has also refactored some test code for improved readability and added additional test cases to ensure the bug is fixed and will not reoccur in the future:

```java
if (set instanceof NavigableSet) {
            final NavigableSet<E> navigableSet = (NavigableSet<E>) set;
            assertThrows(UnsupportedOperationException.class, () -> navigableSet.pollFirst());
            assertThrows(UnsupportedOperationException.class, () -> navigableSet.pollLast());
        }
```



## 3 Ingénierie du Chaos chez Netflix :

### Expériences Réalisées
Netflix met en oeuvre plusieurs expériences pour tester la résilience de ses systèmes :
1. **Chaos Monkey** : Arrêt aléatoire de machines virtuelles en production pour tester la tolérance aux pannes.
2. **Chaos Kong** : Simulation de la panne d’une région entière d’Amazon EC2 pour évaluer les capacités de basculement.
3. **Tests d’injection de pannes** : Échec intentionnel des requêtes entre services pour vérifier la dégradation du système.
4. **Tests de latence** : Ajout de délais dans les communications pour mesurer les impacts sur la performance.

### Conditions Nécessaires
Ces tests se déroulent en production, nécessitant des systèmes d'observation robustes et des mécanismes de sécurité pour limiter les risques. Les métriques comme le nombre de démarrages de flux vidéo par seconde (SPS) sont utilisées comme indicateurs de la santé du système.

### Variables Observées
Netflix observe principalement :
- Les **métriques globales**, comme le SPS, qui reflètent le comportement global du système.
- Les **métriques locales**, comme la latence ou l’utilisation CPU, pour détecter des problèmes internes sans impact direct pour l'utilisateur.

### Résultats Obtenus
Les expériences ont démontré une amélioration de la résilience et de la disponibilité des systèmes. Elles encouragent également une culture de développement où les pannes sont prises en compte dès la conception.

### les Autres Entreprises
Netflix n’est pas seul à appliquer ces techniques. Des entreprises comme Amazon, Google, Facebook ou Microsoft utilisent des approches similaires pour tester la robustesse de leurs infrastructures, notamment par des simulations de pannes massives.

### Applications dans d’Autres Contextes
Ces pratiques peuvent s’adapter à différents domaines, comme l'**E-commerce** ou la **Santé**
- **E-commerce** : Simuler des pics de trafic pour tester la capacité des systèmes à répondre à la demande.
- **Santé** : Simuler des pannes dans des systèmes critiques pour évaluer la gestion des données médicales sensibles.


## 4. WebAssembly

### the main advantages of having a formal specification for WebAssembly:

#### 1. **Sécurité Renforcée** :
   - Assure une exécution sûre, même pour du code provenant de sources non fiables.
   - Garantit l’isolation mémoire et la protection contre les comportements indéfinis, empêchant ainsi les corruptions système.

#### 2. **Performance Optimale** :
   - Permet une validation rapide du code via une vérification des types en une seule passe.
   - Facilite la compilation efficace vers des instructions machine natives, réduisant la latence et maximisant les performances.

#### 3. **Interopérabilité et Portabilité** :
   - Supporte une large gamme d’architectures matérielles et de plateformes logicielles.
   - Établit des bases solides pour le partage de modules et l’intégration dans divers contextes (navigateurs, systèmes embarqués).

#### 4. **Robustesse et Fiabilité** :
   - Simplifie la détection des erreurs en définissant des règles strictes sur la structure et le flux de contrôle du code.
   - Évite les erreurs dues à des sauts ou des boucles mal formés.

#### 5. **Base pour les Optimisations** :
   - La spécification sert de fondation pour des techniques avancées de compilation et d’optimisation, telles que la traduction directe en SSA (Static Single Assignment) pour des compilateurs JIT modernes.


### In your opinion, does this mean that WebAssembly implementations should not be tested?
Non, je pense qu'il devrait être testé de manière approfondie. Le fait qu'il nous permette d'améliorer la performance et la sécurité de notre application web ne signifie pas qu'il soit exempt de tout type de bugs. Au contraire, nous devrions le tester intensivement, précisément parce qu'il est censé améliorer la sécurité et la performance de l'application.






