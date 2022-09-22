
## internal optimization
Es kam mal wieder das Thema `$array += $something` auf. Inzwischen sollte weitgehend bekannt sein, dass es böse ist. Es steht sogar im Handbuch.
[Aufklärung](https://learn.microsoft.com/en-us/powershell/scripting/dev-cross-plat/performance/script-authoring-considerations?view=powershell-7.2#array-addition)  
Kurze Erklärung:  
Arrays sind Immutable (unveränderbar). Also wird in jeder Runde aus dem Array und dem hinzuzufügendem Objekt ein neues Array erstellt.
Interaktiv mit einer Handvoll Objekten fällt das kaum ins Gewicht, bei mehr Objekten wird es aber schnell problematisch.  
(Ausserdem zeigt es mangelndes Verständnis darüber wie Powershell funktionert. Nutzt die Pipeline )

Zum Thema:
Die Benchmarks wurden mit `Measure-Command { <# code to Bench #> }` gemacht und ich erinnerte mich an einen Kommentar von [SeeminglyScience](https://github.com/SeeminglyScience) dass in dem Test die Optimierung fehlt. Die Lösung ist es den zu testenden Code in einen Scriptblock zu packen. `Measure-Command { { <# code to Bench #>} }`  
Ein paar Fehlversuche später habe ich noch mal nachgefragt und [diese Benchmarks](https://gist.github.com/Mettcon/0e333fde41345e1e875289bf4828995a) () brachten das erwartete Ergebnis.  
Die Optimierungen betreffen also hauptsächlich Variablen in fremden Scopes. Daher die übergaben. `{ $emptyArray = $emptyArray; $Iterations = $Iterations;...}`  

Unoptimiert wird jeder Variablenzugriff mit `VariableOps.GetVariableValue` durchgeführt. Womit die Variablen über mehrere Scopes gesucht werden.  
Eine lokale Variable stattdessen ist nur ein Property Zugriff einer internen Klasse.  
Meine große Erkenntnis hier war, dass das nicht nur für `Measure-Command` gilt, sondern für Powershell generell.  
Der Performance Gewinn der Optimisierung wird fast immer nur marginal sein, aber es ist Performance for free, wenn man sich an guten Stil hält, überraschungen vermeidet und evtl. sogar Schwierigkeiten mit dem Strictmode vermeidet. 