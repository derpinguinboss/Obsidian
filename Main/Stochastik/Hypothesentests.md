

Vorgehen
===

_$H_0$_ und $H_1$ bestimmen
---

Wenn man eine Ausgangshypothese hat, wählt man $H_0$ ([[Nullhypothese H0]]) so, dass sie die Gegenhypothese bildet, denn $H_0$ soll verworfen werden, damit die Ausgangshypothese als [[Alternativhypothese H1]] angenommen (**nicht bestätigt**) werden kann.  

Dabei gibt es verschiedene Arten von Hypothesentests: 
1. Einseitige Tests
	1. Linksseitiger Test
	2. Rechtsseitiger Test
2. Zweiseitige Tests

**Beispiel:**
Man möchte bestätigen, dass $p \leq 70\%$.
Man wählt daher (Rechtsseitig): 
$$
\begin{flalign}
H_1 \leq 0.7 \land H_0 > 0.7
&&\end{flalign}
$$
$$
\begin{flalign}
a: min(P(x \leq a) > \alpha)
&&\end{flalign}
$$
$$
\begin{flalign}
VB[0;a] \space \land \space AB[a;n]
&&\end{flalign}
$$

[[Annahmebereich|AB]] und [[Verwerfungsbereich|VB]] mit [[Sigma Regeln]] bestimmen
---

**Gegeben:**
- Test ist Linkseitig.
- $p=0.11; \space n=1000; \space \alpha=5\%$

**Rechnung:**
$\Rightarrow \mu = 110; \space \sigma = 9.89$
$\Rightarrow$ [[Laplace-Bedingung]] erfüllt 
Sigma regeln $\rightarrow$ $P(x \in [\mu-1,64\sigma; \mu+1,64\sigma]) = 0.9$
$\Rightarrow$ $P(x \in [\mu-1,64\sigma; \infty ]) = 0.95$

$\mu-1,64\sigma = 93,77 \approx 94$ (Aufrunden)

$\Rightarrow AB[94;1000]$ $\land$ $VB[0;93]$


[[Annahmebereich|AB]] und [[Verwerfungsbereich|VB]] mit GTR genau bestimmen
---

**Gegeben:**
- Test ist Linkseitig.
- $p=0.11; \space n=1000; \space \alpha=5\%$

$a: P(x \leq a) > \alpha$

GTR: $b(k) := binomcdf(1000, 0.11, 0, k)$

| $a$ | $P(x \leq a)$ |
| ---- | ---- |
| $93$ | $0.045...$ $<$ $0.05$ |
| 94 | $0.056...$ $>$ 0.05 |

$\Rightarrow AB[94;1000]$ $\land$ $VB[0;93]$

