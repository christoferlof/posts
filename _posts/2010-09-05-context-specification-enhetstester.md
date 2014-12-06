---
title: Context/Specification enhetstester
layout: post
tags: agile, testing, swedish
excerpt: Context/Specification är en alternativ stil att skriva enhetstester enligt. Tycker du att ni har enhetstester som är komplexa eller svåra att underhålla av andra anledningar? Då kan Context/Specification stilen vara ett bra verktyg i arbetet med att reda ut situationen.
---
Med stor sannolikhet är det vanligast att enhetstester skrivs enligt den sk _arrange, act, assert_ stilen. Stilen går ut på att först etablera scenen för vad som skall utspela sig (arrange), därefter utförs handlingen (act), för att till sist försäkra sig om att det gick som man hade föreställt sig (assert). Detta sker vanligen inom ramen för en testmetod.

```csharp
[TestMethod]
public void ServiceReturnsFoo() {
   //arrange
   var service = new FooService();
   var expected = "Foo";
            
   //act
   var actual = service.Foo();

   //assert
   Assert.AreEqual(expected,actual);
}
```
