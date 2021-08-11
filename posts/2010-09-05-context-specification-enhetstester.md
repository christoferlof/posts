---
title: Context/Specification enhetstester
layout: post
tags: agile, testing, swedish
excerpt: Context/Specification är en alternativ stil att skriva enhetstester enligt. Tycker du att ni har enhetstester som är komplexa eller svåra att underhålla av andra anledningar? Då kan Context/Specification stilen vara ett bra verktyg i arbetet med att reda ut situationen.
---
# Context/Specification enhetstester

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

Nu och då träffar jag utvecklare som anser enhetstester bidrar till fler problem istället för att avhjälpa eller förhindra dem.

> Testerna går sönder för lätt när systemet under test förändras. När vi skall reparera testerna vet ingen riktigt vad de faktiskt testar - testerna är för komplexa. Eftersom vi inte riktigt förstår testerna så litar vi inte på dem. Ibland är allt grönt, ibland är det lite "halvgrönt". Skall vi lyckas med enhetstester känns det som vi behöver tester som testar testerna.

I takt med att antalet tester ökar ökar också kraven på att testerna är minst lika underhållsvänliga och läsbara som den kod de avser att testa.

Att testerna går sönder lätt kan bero på flera saker. Ett vanligt problem är att testerna är överspecificerade och använder mockramverk allt för intensivt för att simulera interna beroenden. När testerna väl går sönder, eller behöver justeras av andra anledningar, så gäller det som alltid att få ställen har påverkats och därmed behöver ändras. Det handlar helt enkelt om att minimera duplicerad kod i testerna. Det är inte alls ovanligt att två tester har snarlika eller rent av samma arrange och act block för att sedan bara skilja sig åt i assert blocket.

```csharp
//arrange for test 1
var repository = new Mock<IFooRepository>();
.. a few lines of mock setup goes here ..
var service = new FooService(repository);

//arrange for test 2
var repository = new Mock<IFooRepository>();
.. here comes the mock setup again ..
var service = new FooService(repository);
```

För att snabbt förstå vad ett test verifierar så bör namnet på testet vara tydligt samt att antalet asserts hålls så nära ett som möjligt. Ett BazTest med tre asserts är ett exempel på ett dåligt testfall.
I många fall vill man försäkra sig om att flera förändringar har skett i tillståndet av systemet under test och då krävs flera assert uttryck. Två uttryck blir snart tre som blir fyra som blir... Då blir frågan snart *vad är det som faktiskt försäkras och testas?* För att tydliggöra det så ger man möjligen testmetoden ett långt fint namn eller kryddar på med kommentarer i testmetoden. Men vad man då fortfarande går miste om är den visualisering som resultatet av en testsvit faktiskt kan ge. Om en assert av fyra i ett testfall inte går igenom - vad innebär egentligen det? Vad är det som inte fungerar? Vissa saker fungerar uppenbarligen eftersom tre asserts accepterades. Borde inte resultatet bli halvgrönt åtminstone? Frågorna blir många. Kanske måste man till och med debugga testfallet för att verkligen få reda på vad som inte fungerar som planerat.

```csharp
[TestMethod]
public void BazTest() {
   //arrange
   var service = new FooService();
           
   //act
   var result = service.Baz();

   //we need some asserts.. right?
   Assert.IsNotNull(result);   
   Assert.IsTrue(result.Succeeded);
   Assert.AreEqual("42", result.TheMessage);
}
```

Context/Specification stilen kan hjälpa dig skriva stabilare och tydligare tester genom att på ett naturligt sätt bryta ut scenen samt att hålla nere antal asserts till en per testmetod.
Stilen innebär dock att varje scen (i detta fall kallad *sammanhang* eller *context*) får en egen dedikerad klass vilket till en början kan uppfattas som överflödigt. Genom att gruppera klasser i varandra och möjligheten till att ärva ett sammanhang från ett annat sammanhang gör att man snart ändrar uppfattning om att många klasser skulle vara ett problem.

För att anamma Context/Specification stilen behövs en basklass som testfallen ärver från. Basklassen, vilken förslagsvis kallas ContextSpecification, exponerar tre metoder som testfallet kan överrida och därmed nyttja. Metoden Context motsvarar arrange blocket - det vill säga här etableras sammanhanget och scenen för systemet under test riggas upp. Because innehåller troligen endast en rad kod - den rad som förändrar tillståndet i systemet under test. Tänk varför förändras tillståndet? Jo - därför... så har du en förklaring till namnvalet. Båda dessa metoder anropas i basklassen i en metod som är märkt med attributet TestInitialize (Varierar beroende på val av testramverk). Det innebär att metoderna Context och Because exekveras innan varje testmetod för att på så vis återställa sammanhanget och tillståndet till dess utgångspunkt för att sedan exekvera testet.

Varje testmetod är därefter endast ett assert-uttryck som försäkrar att resultatet av tillståndsförändringen är vad som förväntas.

```csharp
[TestClass]
public class when_foo_is_invoked : ContextSpecification {

   protected FooService    Service;
   protected string        Result;

   protected override void Context() {
       var repository = new Mock<IFooRepository>();
       .. mock setup code goes here ..
       Service = new FooService(repository);
   }

   protected override void Because() {
       Result = Service.Foo();
   }

   [TestMethod]
   public void should_return_foo() {
       Assert.AreEqual(expected: "Foo", actual: Result);
   }
}
```

Den riktiga styrkan med Context/Specification stilen är som tidigare nämnt dess enkelhet att ärvas och återanvändas.
Vi tittar lite närmare på ett exempel. 
En tjänst som skapar kontakter tar emot ett meddelande innehållandes kontaktens förnamn, efternamn och e-postadress. Samtliga dessa egenskaper är obligatoriska för att en ny kontakt skall få sparas ned i systemet. I det fall någon av de nämnda egenskaperna på kontakten inte uppfyller valideringsreglerna blir svaret på meddelandet false. 
För att testa alla kombinationer i ovan scenario med arrange-act-assert modellen finns risken att duplicerad skapas genom tre testmetoder vilka innehåller snarlika arrange block och identiska act och assert block.

Med Context/Specification stilen skapas förslagsvis en basklass för sammanhanget där alla kontaktens egenskaper initialiseras till ett godkänt tillstånd. För varje egenskap på meddelandet skapas ytterligare en klass som ärver från grundsammanhanget och som endast justerar aktuell egenskap till ett otillåtet värde. På så vis kan en testsvit skapas med god täckningsgrad, god läsbarhet och god underhållbarhet. Notera hur ett arrange block (i form av Context) och en assert resulterar i tre testfall (testmetoden är del av basklassen).

```csharp
public class when_validating_contact : ContextSpecification {

   protected ContactService    Service;
   protected bool              Result;
   protected Contact           Contact;

   protected override void Context() {
           
       Service = new ContactService();
       //initialize the contact with known, valid, values
       Contact = new Contact {
           Email       = "e-mail",
           FirstName   = "fist-name",
           LastName    = "last-name"
       };
   }

   protected override void Because() {
       Result = Service.Validate(Contact);
   }

   [TestClass]
   public class with_empty_email : when_validating_contact {
       protected override void Context() {
           base.Context();
           //set email to an invalid value
           Contact.Email = string.Empty;
        }
   }

   [TestClass]
   public class with_empty_firstname : when_validating_contact {
       protected override void Context() {
           base.Context();
           //set firstname to an invalid value
           Contact.FirstName = string.Empty;
       }
   }

   [TestClass]
   public class with_empty_lastname : when_validating_contact {
       protected override void Context() {
           base.Context();
           //set lastname to an invalid value
           Contact.LastName = string.Empty;
       }
   }

   [TestMethod]
   public void should_return_false() {
       Assert.IsFalse(Result);
   }
}
```

Ordet *specification* används istället för *test* för att skapa en tydligare upplevelse av ett det är exekverbara specifikationer som skapas. Själv ser jag en klar fördel med att åtminstone använda ett språk i namngivning av *contexts* och *specifications* som ligger så nära som möjligt det språk som används av kunden i ursprunglig kravställning. Man tenderar då få tester (specifikationer) som tydliggör varför klasser finns och hur deras beteende associerar till kundens önskemål. Att gruppera testfallen efter *feature* eller *user story* kan stärka spårbarheten ytterligare.

`[Pass] When_validating_contact+with_empty_name.should_not_be_valid` är enligt min mening ett mer tydligt testresultat än `[Pass] BazTest`. Kanske förstår också din kund vad som faktiskt fungerar av det hon efterfrågar.
