---
layout: post
published: false
tags:
  - angular
title: Dynamiczne tworzenie kontrolek w Angular 2
---

Standardowo aplikację budujemy korzystając z komponentów używających komponentów, które używają komponentów, i tak dalej...

Komponenty określają selektory którymi możemy je osadzać oraz szablony HTML opisujące sposób prezentacji. Korzystając z tego zestawu, w kolejnych szablona osadzamy kolejne komponenty wykorzystując ich selektory, zupełnie jakby były to natywne elementy HTMLa.

Co jednak, jeżeli nie jesteśmy w stanie ustalic konkretnego komponentu na etapie pisania aplikacji, a dopiero w trakcie jej wykonania?

## Szybkie rozwiązanie

Pierwsze co może nam przyjść do głowy to wykorzystanie _ngIf_ i opisanie widoku w formie frontendowego _switch'a_.

Spójrzmy na przykład:

```javascript
@Component({
  	selector: 'app',
    template: `
      	<div>
        	<book-details  *ngIf="model.type === 'book'"></book-details>
            <movie-details *ngIf="model.type === 'movie'"></movie-details>
            <comic-details *ngIf="model.type === 'comic'"></comic-details>
        </div>
    `
})
class AppComponent {}
```
    
Na pierwszy rzut oka wszystko wygląda nieźle, w zależności od typu prezentowanego obiektu potrafimy wyświetlić odpowiedni komponent prezentujący szczegóły. Nawet jesteśmy z siebie zadowoleni, w końcu mamy komponenty, a przecież mogliśmy zaszyć prezentację typów bezpośrednio w szablonie :wink:

Spójrzmy jednak krytycznie na nasz twór, a zauważymy potencjalne problemy.

Po pierwsze, dodanie nowego typu komponentu za każdym razem wiąże się z modyfikacją wszystkich szablonów komponentów zależnych. Dodając nową funkcjonalność do systemu będziemy musieli zmienić wiele, teoretycznie niezależnych, fragmentów kodu. W praktyce podążając tą drogą szybko dotrzemy do _wzorca Copy'ego i Paste'a_. Stąd już blisko żeby trafić na projektowy _wall of shame_ za klasyczny [_Shotgun surgery_](https://en.wikipedia.org/wiki/Shotgun_surgery).

Kolejny, być może nawet bardziej narzucający się, problem to wypłynięcie warstwy logiki do warstwy prezentacji. Mieszanie logiki z prezentacją to nigdy nie jest dobry pomysł. W szczególności w świecie frontendu, gdzie szukanie referencji czy refaktoryzacja pomiędzy HTMLem a JS/TSem to zawsze loteria.

## Lepsze podejście

Co gdybyśmy mogli przenieść logikę wyboru komponentu do klasy? Dodatkowo w szablonie wskazać gdzie komponent ma zostac wyrenderowany? Nadal nie tracąc niczego z komponentowego podejścia, w tym wstrzykiwania zależności? Dokładnie tak możemy to zrobić :wink:

W pierwszym kroku przerobimy szablon komponentu:

```javascript
@Component({
	selector: 'app',
	template: `
		<div>
			<div #details></div>
		</div>
	`
})
class AppComponent {}
```

Dotychczasowe definicjie konkretnych komponentów zastępujemy pojedynczym _div'em_ pełniącym funkcję placehodlera. Dodatowo oznaczamy go jako zmienną lokalną o nazwie _details_. Dzięki temu w kolejnym kroku będziemy mogli odnieść się do niego z kontrolera komponentu.

```javascript
@Component(...)
class AppComponent {
	@ViewChild('details', {read: ViewContainerRef})
	private placeholder: ViewContainerRef;
}
```

Stosując dekorator _@ViewChild_ wstrzykujemy element _DOMu_ do kontrolera komponentu. Pierwszy parametr może przyjąć klasę oczekiwanego elementu lub selektor. Dodatkowo przekazujemy jakiej klasy element nas interesuje. W przypadku wstrzykiwania po selektorze standardowo jest to obiekt klasy _ElementRef_, nas jednak interesuje obiekt typu _ViewContainerRef_.

Klasa _ViewContainerRef_ jest o tyle ciekawa, że dostarcza metodę pozwalającą tworzyć komponenty w locie na podstawie dostarczonej fabryki:

```javascript
createComponent(
	componentFactory: ComponentFactory<C>, 
    index?: number, 
    injector?: Injector, 
    projectableNodes?: any[][]
) : ComponentRef<C>
```

Zmienna oznaczona dekoratorem _@ViewChild_ zostanie uzupełniona w trakcie tworzenia widoku, to znaczy że możemy się nią posługiwać dopiero w fazie _ngAfterViewInit_. W efekcie możemy napisać kolejny kawałek naszego komponentu:

```javascript
@Component(...)
class AppComponent {

	@ViewChild('details', {read: ViewContainerRef})
	private placeholder: ViewContainerRef;
    
    ngOnInit():void {
    	this.detailsComponentRef = this.placeholder.createComponent(this.detailsComponentFactory);
    }
    
}
```

Teraz pozostaje nam już tylko uzyskanie fabryki komponentów. W tym celu należy...

todo:
- przekazywanie wartości do stworzonego komponentu
- nasłuchiwanie na zmiany stworzonego komponentu
- konieczność dodania nowej własności w definicji modułu i z czego to wynika
- skąd wziąć fabrykę

Materiały:
- [Komunikacja pomiędzy komponentami, w tym oznaczanie jako zmienne lokalne i odwołania z kontrolerów.](https://angular.io/docs/ts/latest/cookbook/component-communication.html#!#parent-to-child-local-var)
- [Dokumentacja dekoratora _@ViewChild_](https://angular.io/docs/ts/latest/api/core/index/ViewChild-decorator.html)
- [Dokumentacja klasy _ViewContainerRef_](https://angular.io/docs/ts/latest/api/core/index/ViewContainerRef-class.html)
- [Cykl życia komponentu](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html)
