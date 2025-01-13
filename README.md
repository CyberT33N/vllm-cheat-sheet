# vllm-cheat-sheet



# Install

<br><br>

## Ubuntu
```shell
uv venv myenv --python 3.12 --seed
source myenv/bin/activate

uv pip install vllm
```











<br><br>
<br><br>
___
<br><br>
<br><br>


# Models

<br><br>

## Supported Models
- https://docs.vllm.ai/en/latest/models/supported_models.html#supported-models
- vLLM supports generative and pooling models across various tasks. If a model supports more than one task, you can set the task via the --task argument. For each task, we list the model architectures that have been implemented in vLLM. Alongside each architecture, we include some popular models that use it.

<br><br>
<br><br>

## Load Model
```
from vllm import LLM

# For generative models (task=generate) only
llm = LLM(model=..., task="generate")  # Name or path of your model
output = llm.generate("Hello, my name is")
print(output)

# For pooling models (task={embed,classify,reward,score}) only
llm = LLM(model=..., task="embed")  # Name or path of your model
output = llm.encode("Hello, my name is")
print(output)
```













<br><br>
<br><br>
___
<br><br>
<br><br>


# Quantization
https://docs.vllm.ai/en/latest/features/quantization/index.html


### **Erklärung der Quantisierungsarten in vLLM**

Quantisierung ist ein Verfahren, um die Speicher- und Rechenanforderungen eines Modells zu reduzieren, indem weniger präzise Zahlenformate (z. B. 8-Bit statt 32-Bit) verwendet werden. Das Ziel ist, die Leistung zu verbessern (z. B. schnellere Inferenz, weniger Speicherverbrauch) bei minimalem Verlust der Genauigkeit. Hier sind die wichtigsten Methoden, die in vLLM unterstützt werden:

---

### **1. AutoAWQ (Automatic Optimal Weight Quantization)**
- **Was es macht**: Optimiert die Gewichte des Modells automatisch, um die Genauigkeit bei der Quantisierung zu maximieren.
- **Vorteil**: Erreicht oft bessere Ergebnisse als einfache Quantisierungsmethoden (wie FP8 oder INT8), da es Gewichtsanpassungen intelligent durchführt.
- **Wann nutzen?**: Wenn du hohe Genauigkeit bei geringem Speicherverbrauch benötigst, ohne manuell optimieren zu müssen.

---

### **2. BitsAndBytes**
- **Was es macht**: Implementiert Low-Bit-Quantisierung (z. B. 4-bit, 8-bit) für Modelle mit minimalem Rechenaufwand.
- **Vorteil**: Sehr effiziente und weit verbreitete Methode, da sie flexibel auf verschiedenen Hardware-Systemen einsetzbar ist.
- **Wann nutzen?**: Für allgemeine Einsätze auf GPUs mit begrenztem Speicher, besonders bei großen Modellen.

---

### **3. GGUF**
- **Was es macht**: Ein Format für quantisierte Modelle, speziell für Tools wie Llama.cpp optimiert.
- **Vorteil**: Läuft effizient auf CPUs und ressourcensparenden GPUs. Ideal für Deployment in kostengünstigen Umgebungen.
- **Wann nutzen?**: Wenn du vorquantisierte Modelle für Hardware mit geringer Leistung brauchst.

---

### **4. INT8 W8A8**
- **Was es macht**: Reduziert die Präzision von Gewichten (W) und Aktivierungen (A) auf 8-Bit (Integer 8).
- **Vorteil**: Spart Speicherplatz und erhöht die Rechengeschwindigkeit, besonders auf GPUs mit INT8-Unterstützung.
- **Wann nutzen?**: Für schnelle Inferenz auf moderner Hardware wie NVIDIA GPUs (mit Tensor Cores).

---

### **5. FP8 W8A8**
- **Was es macht**: Verwendet 8-Bit Floating-Point (FP8) für Gewichte und Aktivierungen.
- **Vorteil**: Kombiniert die Vorteile von geringer Speichergröße mit der Flexibilität von Floating-Point-Berechnungen, was oft höhere Genauigkeit als INT8 bringt.
- **Wann nutzen?**: Für moderne GPUs (z. B. NVIDIA H100), die FP8 effizient unterstützen.

---

### **6. FP8 E5M2 KV Cache**
- **Was es macht**: Speichert den Key-Value-Cache (der für längere Kontextlängen benötigt wird) in FP8 mit dem Format E5M2 (5 Bits für Exponent, 2 Bits für Mantisse).
- **Vorteil**: Spart Speicherplatz im Cache und ermöglicht längere Sequenzen, ohne viel Speicher zu benötigen.
- **Wann nutzen?**: Wenn du längere Sequenzen verarbeiten musst und Speicherplatz sparen willst.

---

### **7. FP8 E4M3 KV Cache**
- **Was es macht**: Ähnlich wie E5M2, aber verwendet 4 Bits für den Exponenten und 3 Bits für die Mantisse. Noch sparsamer im Speicherverbrauch.
- **Vorteil**: Extrem niedriger Speicherverbrauch für den KV-Cache, allerdings mit potenziell größerem Genauigkeitsverlust.
- **Wann nutzen?**: Für Anwendungen, bei denen Speicherknappheit die höchste Priorität hat.

---

### **Vergleich der Methoden**

| **Methode**       | **Speichereffizienz** | **Genauigkeit**   | **Hardwarebedarf**         | **Einsatzbereich**                     |
|--------------------|-----------------------|-------------------|----------------------------|----------------------------------------|
| **AutoAWQ**        | Mittel               | Sehr hoch         | Flexibel                  | Präzise Anwendungen mit geringer Optimierungsarbeit |
| **BitsAndBytes**   | Hoch                 | Hoch              | Flexibel                  | Standardlösung für GPUs               |
| **GGUF**           | Sehr hoch            | Mittel            | CPUs/Low-End-GPUs         | Kostengünstige Inferenz               |
| **INT8 W8A8**      | Hoch                 | Mittel            | INT8-fähige GPUs          | Schnelle und effiziente Inferenz      |
| **FP8 W8A8**       | Hoch                 | Hoch              | FP8-fähige GPUs (z. B. H100) | Hochleistungsanwendungen             |
| **FP8 E5M2 Cache** | Sehr hoch            | Hoch              | Modern GPUs               | Lange Sequenzen mit gutem Speicherverbrauch |
| **FP8 E4M3 Cache** | Extrem hoch          | Mittel bis niedrig| Modern GPUs               | Maximale Speicheroptimierung          |

---

### **Zusammenfassung**
- **Für Anfänger und unkomplizierte Nutzung**: **GGUF** oder **BitsAndBytes**.
- **Für optimierte Performance auf moderner Hardware**: **FP8 W8A8** oder **AutoAWQ**.
  - Die RTX 4090 unterstützt keine FP8 nativ (nur neuere GPUs wie H100). Daher ist FP8 für dich keine Option. 
- **Wenn Speicher im Fokus steht**: **INT8 W8A8** oder FP8-KV-Cache-Methoden.

Die Wahl hängt von deiner Hardware und deinen Anforderungen an Speicher und Genauigkeit ab!
