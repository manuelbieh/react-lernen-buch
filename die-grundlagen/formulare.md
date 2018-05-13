# Formulare

Formulare besitzen in React eine kleine Sonderstellung und funktionieren etwas anders als andere DOM-Elemente, da Formulare eine Art **eigenen State** besitzen, der erst einmal nichts mit dem React-State gemein hat. 

Der State von Textfeldern besteht bspw. aus dem eingegebenen Wert, der State von Checkboxen oder Radiobuttons resultiert aus der Tatsache ob diese ausgewählt sind oder nicht, Auswahllisten \(`<select></select>`\) halten als State den ausgewählten Wert bzw. bei Mehrfachauswahl die ausgewählten Werte. React ändert an diesem Verhalten grundsätzlich erstmal nichts. Wer möchte, kann das so beibehalten und muss sich um nichts weiter kümmern. 

Im React-Jargon ist dann die Rede von **Uncontrolled Components**, also **unkontrollierten Komponenten**. Unkontrolliert deshalb, weil React sich nicht um das State-Management dieser Komponenten kümmert. Das State-Handling ist entweder vollständig unabhängig von React oder funktioniert nur **unidirektional** aus Richtung der DOM Formular-Elemente hin zum React State, jedoch **nicht in die entgegengesetzte Richtung**. Von einem Update am React-State bekommt ein Formular-Element also nichts mit und zeigt weiter den gleichen Wert \(oder Status bei Checkboxen und Radiobuttons\) wie zuvor.

Demgegenüber stehen die **Controlled Components**, also **kontrollierte Komponenten**. Diese sind bidirektional, hier aktualisiert also ein Update am React-State den internen Status des Formular-Elements und ebenso aktualisiert ein Update am jeweiligen Formular-Element den React-State in die entgegengesetzte Richtung. **Controlled Components** sind etwas aufwändiger in der Implementierung, sind zugleich jedoch auch die „sicherere“ Variante, da wir nicht Gefahr laufen, dass beide States voneinander abweichen.

## Uncontrolled Components / unkontrollierte Komponenten

**Unkontrollierte Komponenten** können dabei im Wesentlichen in 2 verschiedenen Formen auftreten. Bei der ersten Variante werden einfach nur Formular-Elemente gerendert, die beim Abschicken bspw. rein serverseitig verarbeitet werden und in keiner Weise mit React interagieren. Ein komplett statisches Formular wenn man so will. React kümmert sich dabei **nicht von alleine** um die Anbindung an den React-State sondern **lässt dem Entwickler hier sämtliche Freiheiten**!

Bei der zweiten Variante werden Änderungen an einem Formular-Element **in den React-State** geschrieben um bspw. im Hintergrund eine Validierung der Daten vorzunehmen oder die eingegebenen Daten an anderer Stelle auszugeben. Eine Änderung am React-State an anderer Stelle der Anwendung hat dabei keinerlei Einfluss auf die Formularfelder. Eben unidirektional, nur in eine Richtung. 

Beispiel für eine solche unkontrollierte Komponente: 

```jsx
class Uncontrolled extends React.Component {
  state = {};

  changeUsername = e => {
    const { value } = e.target;
    this.setState(() => ({
      username: value,
      isValid: value.length > 3,
    }));
  };

  submitForm = e => {
    e.preventDefault();
    alert(`Hallo ${this.state.username}`);
  };

  render() {
    const { isValid, username } = this.state;
    return (
      <form method="post" onSubmit={this.submitForm}>
        <p>Dein Benutzername: {username}</p>
        <p>
          <input type="text" name="username" onChange={this.changeUsername} />
          <input type="submit" disabled={!isValid} />
        </p>
      </form>
    );
  }
}
```

Hier sehen wir ein einfaches Textfeld in das der Benutzer einen gewünschten Benutzernamen eintragen kann. Die `Uncontrolled` Komponente wird mittels `onChange`-Event von jeder Änderung in Kenntnis gesetzt und kann den Benutzernamen weiterverarbeiten. Da React hier nur **passiv** agiert, also bei einer Änderung am Textfeld über den neuen Wert in Kenntnis gesetzt wird, bewegen wir uns immer noch im Bereich der **Uncontrolled Components**.

Dies ist in einigen Fällen  ausreichend, insbesondere wenn die Formulare noch nicht all zu komplex sind. Allerdings ist der React State hier vom DOM State **entkoppelt** bzw. funktioniert nur **in eine Richtung**. Der React State wird aktualisiert sobald der `onChange`-Event des Textfelds ausgelöst wird. Allerdings bedeutet dies, dass nicht gleichzeitig auch unser Textfeld aktualisiert wird wenn der Wert im React State an anderer Stelle verändert wurde, bspw. weil der Response eines asynchronen Requests nach einiger Zeit eintrifft.

## Controlled Components / kontrollierte Komponenten

Um sowohl State-Updates in Formular-Feldern zu abzubilden als auch auf der anderen Seite benutzerseitige Änderungen an Formularfeldern in den React-State zu übertragen, benötigen wir eine **Controlled Component**. Hier überlassen wir das State-Handling eines Formular-Elements vollständig React. Dies bedeutet, dass wir das `value`-Attribut mit einem Wert befüllen den wir aus dem React-State beziehen und gleichzeitig auch einen geänderten Wert wieder zurück in den React-State überführen.

Das Ziel bei diesem Ansatz ist es, den React-State als **Single Source of Truth** zu betrachten, also als die einzige Quelle der Wahrheit. Relevant ist der Wert der im React-State steht, das jeweilige Eingabefeld reflektiert dann zu jedem Zeitpunkt den Wert aus dem React-State.

Schauen wir uns auch hier mal ein Beispiel an:

```jsx
class Controlled extends React.Component {
  state = {
      username: '',
      isValid: false,
  };

  submitForm = e => {
    e.preventDefault();
    alert(`Hallo ${this.state.username}`);
  };

  changeUsername = e => {
    const { value } = e.target;
    this.setState(() => ({
      username: value,
      isValid: value.length > 3
    }));
  };

  render() {
    const { username } = this.state;
    return (
      <form method="post" onSubmit={this.submitForm}>
        <p>{username}</p>
        <p>
          <input
            type="text"
            name="username"
            onChange={this.changeUsername}
            value={username}/>
          <input type="submit" disabled={!this.state.isValid} />
        </p>
      </form>
    );
  }
}
```

Auf den ersten Blick unterscheidet sich die `Controlled` Komponente gar nicht sonderlich von der `Uncontrolled` Komponente im Absatz oben. Und tatsächlich ist der entscheidende Faktor der die unkontrollierte Komponente zu einer kontrollierten werden lässt das `value`-Attribut im `<input />`-Element. Ist dieses Vorhanden kontrolliert React das Formular-Element und erwartet, dass sich Änderungen am Eingabefeld entsprechend im State widerspiegeln.

Hier gibt es einige Dinge zu beachten. So darf der Wert des `value`-Attributes immer nur ein String sein, niemals `undefined` oder `null`.

![Warnung bei einem kontrollierten Textfeld mit dem value &quot;null&quot;](../.gitbook/assets/react-uncontrolled-null.png)

Eine Ausnahme sind hier `select`-Elemente die gleichzeitig ein `multiple`-Attribut besitzen. Hier muss das `value`-Attribut ein Array sein. Moment mal, denkt ihr euch jetzt vielleicht. Welches value-Attribut beim `<select>`? Optionen selektiere ich doch indem ich das `selected`-Attribut bei der jeweiligen `<option>` setze? Und ja, das ist korrekt in HTML, in React funktioniert das ein klein wenig anders. Hier wird der kontrollierte Wert ebenfalls über das value-Attribut gesetzt. Dasselbe gilt übrigens auch für das `<textarea>`-Element, dessen Initialwert für gewöhnlich durch seinen textContent bestimmt wird.

Nicht so in React. React vereinheitlicht hier den Mechanismus und erfordert für die drei Elemente `input` \(egal welcher `type`\), `textarea` sowie `select` ein `value`-Attribut! Bei einfachen Werten muss dies **immer ein String** sein, bei einer Auswahlliste mit dem `multiple`-Attribut wie eben erwähnt ein **Array bestehend aus Strings**!




