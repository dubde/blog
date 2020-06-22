---
layout: post
title: Logica nei templates
excerpt: I templates hanno la possibilita` di includere sempre piu` logica, ma siamo sicuri sia questa la giusta direzione?
author: Deiv
tags: c#, js, ts, angular, MVC, templates, logic
---

MVC permette di separare business logic, data model e presentation layer attraverso la separazione in diversi contesti e responsabilita1.
Ma quando implementiamo UI complesse abbiamo bisogno di gestire elementi condizionali a seconda dei valori nel data model.
Parlando di MVC con razor files, normalmente questo viene fatto nei files di template cshtml, in testa al file in un blocco di codice dedicato

```
@using DynamicPlaceholders.Mvc.Extensions
@using System.Text;

@{

    var containerTheme = GetRenderingParameter("CssTheme");
    var containerCssClass =GetRenderingParameter("CssClass");
    StringBuilder sb = new StringBuilder();
    if (GetRenderingParameter("Full Width") == "1") { sb.Append("full-width "); }
    if (GetRenderingParameter("Centered") == "1") { sb.Append("centered "); }
}

<div class="component container @containerCssClass @sb.ToString() @containerTheme">
    @Html.Sitecore().DynamicPlaceholder("tcp-container")
</div>
```

cosi` che i designer o front-end developer possano cambiare in tempo reale la visualizzazione senza dover ri-pubblicare l'intero progetto. I files di
template infatti non vengono compilatii ma vengono soltanto interpretati. Sinceramente, la soluzione non mi piace per vari motivi.

1. La logica non e` testabile.
   Se vogliamo essere sicuri che la rappresentazione mostrata sia coerente con le specifiche dobbiamo scrivere un test appropriato. Ma se releghiamo la logica
   al template, non possiamo scrivere uno unit test dedicato.
2. Il codice non e` compilato prima di andare in runtime (.NET)
3. Business logic nascosta
4. template condizionali sono poco leggibili

In che senso questi sono problemi? Partendo da una situazione senza test e di editing diretto, possiamo cadere nella situazione in cui il codice nel non sia coerente con la base, vedi typos o refactoring nel backend. Assumiamo che i membri del modello e i servizi siano accessibili al template ma senza verificarlo. Cosa succede se un servizio nasconde una chiamata alle API e lancia un eccezione? Interfaccia bloccata. Se manca una dipendeza essenziale? Interfaccia bloccata. Debuggare i template non e` pratico, ma soprattutto si perde la separazione tra business logic e presentazione.

Vogliamo finalmente ammettere che gli elementi condizionali nell'esperienza utente sono dettati da bisogni di business coniugati per l'utente? Un esempio pratico ma non banale.

Vogliamo mostrare un pop-up d' errore in rosso e nel caso di warning in arancione. Ux indica il fatto di dover cambiare dei dettagli tra Warning e Errore ma business logic potrebbe influenzare la scelta di quali colori a seconda dell'applicazioneo del prodotto. Ad esempio se viene utlizzata l'interfaccia per un programma specifico per daltonici probabilmente le gradazioni di colori dovranno essere specifiche. Da lato template la struttura e` sempre la stessa, la decisione sul colore riguarda una scelta di ux assieme a business logic.

Difficilmente posso trovare una logica di presentazione non dettata da un bisogno a monte rispetto il template. Anche nel codice di prima, chi dedcide se il layout sia full-width o no non sono specifiche dello schermo ma impostazioni settate dall'utente. Daltronde tutte le limitazioni legate alla piattaforma su cui gira l'interfaccia possono essere supportate da altre tecnologie come i mixins in CSS.

Se mettiamo tra il modello e il template un view-model con dedicato controller possiamo dedicarci alla creazione degli elementi che dovranno semplicemente essere disposti sul template, testabile 100% e compilato con il resto del progetto.

Quindi come posso gestire il rendering, anche condizionale, di singoli membri del data model?

La soluzione, facilmente visibile in Angular, arriva con i Pipes. In Angular, quando vuoi formattare, controllare il rendering di un parametro, puoi ovviamente usare tutti i trucchi anche disponibili in C# per avere codice puro nel template. Ma la soluzione elegante consiste nell'uso di funzioni di pipeline che prendono in ingresso un dato e lo restituiscono formattato. Ad esempio per la gestione delle date, valute, numeri decimali, email sono disponibili di default, ma si possono sempre scrivere funzioni personalizzate a seconda del bisogno.

Tirando le somme, abbiamo visto come scrivere codice nel template non sia necessario ma anzi rischioso e non leggibile, mentre usare strumenti nel codebase possono prevenire errori come fornire un'adeguata e completa documentazione sull'esperienza utente dettata da bisogni specifici.
