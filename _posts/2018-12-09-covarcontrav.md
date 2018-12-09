---
layout: post
title:  "Covariance & Contravariance"
date:   2018-12-09 12:00:00
categories: Programming
tags: Scala Java
---

* content
{:toc}

I've been learning covariance and contravariance concepts in programming recently and tried to make it easy to understand.
> What is covariance?
Type B is a subtype of type A, if you want C[B] to be a subtype of C[A], you should use covariance

> On contrary, if type B is a subtype of type A, if you want C[A] to be a subtype of C[B], you should use contravariance


Following is Scala demo of using covariance and contravariance.
```scala
object Demo {
  def main(args: Array[String]) = {
    val petStore = new PetStore()
    petStore.storePet(new Cage[Pet](new Pet()))
    petStore.storePet(new Cage[Dog](new Dog()))
    petStore.storePet(new Cage[Cat](new Cat()))
    // not allowed
    // petStore.storePet(new Cage[Tiger](new Tiger()))

    val zoo = new Zoo()
    zoo.store(new ZooCage[Tiger](new Tiger()))
    zoo.store(new ZooCage[DangerousAnimal](new DangerousAnimal()))
    zoo.store(new ZooCage[Animal](new Animal()))

    // not allowed
    // zoo.store(new ZooCage[Pet](new Pet()))
  }
}


class PetStore {
  def storePet(cage: Cage[Pet]) = {
    println(s"${cage.toString} stored")
  }
}

class Zoo {
  def store(zooCage: ZooCage[Tiger]) = {
    println(s"${zooCage.toString} caged")
  }
}


class Animal {
  override def toString(): String = {
    "An animal"
  }
}

class DangerousAnimal extends Animal {
  override def toString(): String = {
    "A Dangerous animal"
  }
}

class Pet extends Animal {
  override def toString(): String = {
    "A pet"
  }
}

class Dog extends Pet {
  override def toString(): String = {
    "A puppy"
  }
}

class Cat extends Pet {
  override def toString(): String = {
    "A kitten"
  }
}

class Tiger extends DangerousAnimal {
  override def toString(): String = {
    "A tiger"
  }
}

// covariance
class Cage[+T](animal: T) {
  override def toString: String = {
    animal.toString
  }
}
// contravariance
class ZooCage[-T](animal: T) {
  override def toString: String = {
    animal.toString
  }
}
```

Java also supports such features.
```java
List<? extends A>
List<? super B>
```


