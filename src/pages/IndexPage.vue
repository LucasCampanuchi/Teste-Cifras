<template>
  <div class="container">
    <p>Visualizador de Cifras</p>

    <textarea
      class="chord-input"
      placeholder="Digite sua cifra aqui...
Exemplo:

         D
Te amo, Deus
      G          D
Tua graça nunca falha
          Bm7
Todos os dias"
    ></textarea>

    <div class="controls">
      <label for="transpose-select">Transpor:</label>
      <select id="transpose-select" v-model="transposeValue">
        <option v-for="n in 12" :key="n" :value="n - 6">{{ n - 6 }}</option>
      </select>
    </div>

    <div class="row">
      <button @click="processChords">Processar Cifra</button>
      <button @click="clearAll">Limpar</button>
      <button @click="generatePdf">Gerar PDF</button>
    </div>

    <div class="chord-display"></div>
    <div class="chord-diagram"></div>
  </div>
</template>

<script>
import ChordSheetJS from "chordsheetjs";
import { Chord } from "chordsheetjs";
import { parse } from "chordjs";

import jsPDF from "jspdf";

export default {
  data() {
    return {
      transposeValue: 0, // Valor inicial para transposição
    };
  },
  methods: {
    processChords() {
      const input = document.querySelector(".chord-input").value;
      const display = document.querySelector(".chord-display");
      const diagramArea = document.querySelector(".chord-diagram");

      if (!input.trim()) {
        display.innerHTML =
          '<div class="error">Por favor, insira uma cifra.</div>';
        return;
      }

      // Limpar área de diagramas
      diagramArea.innerHTML = "";

      // Processar o texto e extrair acordes
      const parser = new ChordSheetJS.UltimateGuitarParser();

      const song = parser.parse(input);

      /* song.lines.forEach((line) => {
        line.items.forEach((item) => {
          if (item.chords) {
            item.chords.forEach((chord) => {
              chords.add(chord.name);
            });
          }
        });

      }); */

      song.lines.forEach((line) => {
        line.items.forEach((item) => {
          if (item.chords) {
            item.chords = item.chords
              .split(" ")
              .map((chord) => this.transposeChord(chord, this.transposeValue))
              .join(" ");
          }
          /* if (item instanceof z) {
            item.chords = item.chords
              .split(" ")
              .map((chord) => this.transposeChord(chord, this.transposeValue))
              .join(" ");
          } */
        });
      });

      // Formatar a cifra transposta
      const formatter = new ChordSheetJS.TextFormatter();
      const formattedText = formatter.format(song);

      // Exibir texto processado
      display.textContent = formattedText;
    },
    transposeChord(chord, value) {
      if (chord === "" || chord === " ") return chord;

      console.log(chord);

      const parsedChord = Chord.parse(chord);

      if (!parsedChord) {
        return chord;
      }

      let newChord = "";

      if (parsedChord.root !== null) {
        newChord = parse(parsedChord.root.originalKeyString)
          .transpose(value)
          .toString();
      }

      if (parsedChord.suffix !== null) {
        newChord += parsedChord.suffix;
      }

      if (parsedChord.bass !== null) {
        newChord +=
          "/" +
          parse(parsedChord.bass.originalKeyString).transpose(value).toString();
      }

      return newChord;
    },
    clearAll() {
      document.querySelector(".chord-input").value = "";
      document.querySelector(".chord-display").innerHTML = "";
      document.querySelector(".chord-diagram").innerHTML = "";
    },
    generatePdf() {
      const input = document.querySelector(".chord-input").value;
      const display = document.querySelector(".chord-display").textContent;

      if (!input.trim()) {
        alert("Por favor, insira uma cifra antes de gerar o PDF.");
        return;
      }

      const pdf = new jsPDF();
      const title = "Cifra Transposta";
      const margin = 10;
      const lineHeight = 10; // Ajusta o espaçamento entre linhas
      let yPosition = 20;

      // Configurar título
      pdf.setFont("Courier", "normal");
      pdf.setFontSize(16);
      pdf.text(title, pdf.internal.pageSize.getWidth() / 2, yPosition, {
        align: "center",
      });
      yPosition += 10;

      // Configurar texto da cifra
      pdf.setFontSize(12);
      const lines = display.split("\n");

      lines.forEach((line) => {
        if (yPosition > pdf.internal.pageSize.getHeight() - margin) {
          pdf.addPage();
          yPosition = margin;
        }
        // Usar fonte monoespaçada garante alinhamento uniforme
        pdf.text(line, margin, yPosition);
        yPosition += lineHeight;
      });

      // Salvar PDF
      pdf.save("cifra_transposta.pdf");
    },
  },
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  height: 100vh;
  margin: 0 auto;
  padding: 20px;
  background-color: #f5f5f5;
  display: flex;
  justify-content: center;
  align-items: center;
}

.container {
  width: 100%;
  background-color: white;
  height: 90%;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.chord-input {
  width: 100%;
  height: 150px;
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-family: monospace;
}

.controls {
  margin: 20px 0;
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
}

button {
  background-color: #4caf50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  flex: 1 0 100px;
}

button:hover {
  background-color: #45a049;
}

.chord-display {
  font-family: monospace;
  white-space: pre-wrap;
  padding: 20px;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-top: 20px;
}

.chord-diagram {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-top: 20px;
}

.chord-item {
  text-align: center;
}

.error {
  color: red;
  margin-top: 10px;
}

.row {
  gap: 10px;
}
</style>
