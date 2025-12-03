# C Sharp Konsolenapp Binary Clock

<img width="476" height="490" alt="image" src="https://github.com/user-attachments/assets/59bc2187-63ef-4f01-925a-e9dbabfc46b7" />

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace BinaryLightClock190825
{


        internal class Program
        {
            // --- kleines Zeit-Datenobjekt ---
            struct TimeParts
            {
                public int Hour { get; }
                public int Minute { get; }
                public int Second { get; }
                public TimeParts(int h, int m, int s) { Hour = h; Minute = m; Second = s; }
                public override string ToString() { return string.Format("{0:00}.{1:00}.{2:00}", Hour, Minute, Second); }
            }

            // --- Blockschrift-Glyphen für Ziffern (5 Zeilen hoch, 3 Zeichen breit) ---
            static readonly Dictionary<char, string[]> Glyph = new Dictionary<char, string[]>
            {
                ['0'] = new[] { "███", "█ █", "█ █", "█ █", "███" },
                ['1'] = new[] { "  █", "  █", "  █", "  █", "  █" },
                ['2'] = new[] { "███", "  █", "███", "█  ", "███" },
                ['3'] = new[] { "███", "  █", "███", "  █", "███" },
                ['4'] = new[] { "█ █", "█ █", "███", "  █", "  █" },
                ['5'] = new[] { "███", "█  ", "███", "  █", "███" },
                ['6'] = new[] { "███", "█  ", "███", "█ █", "███" },
                ['7'] = new[] { "███", "  █", "  █", "  █", "  █" },
                ['8'] = new[] { "███", "█ █", "███", "█ █", "███" },
                ['9'] = new[] { "███", "█ █", "███", "  █", "███" },
                ['.'] = new[] { "   ", "   ", "   ", "   ", " █ " },
            };

            // --- Block-"Glyphen" für Bits der Binäruhr (3x3) ---
            static readonly string[] BIT1 = { "███", "███", "███" }; // an
            static readonly string[] BIT0 = { "   ", "   ", "   " }; // aus

            // --- Abstände/Skalierung (einfach anpassen) ---
            const int BIN_COL_GAP = 2; // Spaltenabstand
            const int BIN_GROUP_GAP = 3; // Extra-Abstand nach Spalte 1 und 3 (HH | MM | SS)
            const int BIT_HEIGHT = 3; // vertikale „Skalierung“ je Bitreihe (Zeilen pro Reihe)
            

            static void Main(string[] args)
            {
                Console.OutputEncoding = Encoding.UTF8;
                Console.CursorVisible = false;


                while (true)
                {
                    var now = DateTime.Now;
                    var t = new TimeParts(now.Hour, now.Minute, now.Second);

                    bool[,] grid = EncodeHhMmSsTo6x4(t); // 6 Spalten × 4 Bitreihen (BCD je Ziffer)

                    Console.SetCursorPosition(0, 0);

                    // 1) Binäruhr im Blockstil
                    RenderBinaryBig(grid);
                    Console.ForegroundColor = ConsoleColor.Green;

                // 2) Labels unter der Binärmatrix
                Console.WriteLine();
                    Console.WriteLine("Stunde          Minute       Sekunde");

                    // 3) Große Dezimaluhr im selben Stil
                    Console.WriteLine();
                    RenderBigTime(t.ToString());

                    // driftarme Sekundensynchronisation
                    var next = now.AddSeconds(1);
                    int sleep = (int)Math.Max(0, (next - DateTime.Now).TotalMilliseconds);
                    Thread.Sleep(sleep);
                }
            }

            // --- hhmmss → 6x4 BCD-Matrix (MSB→LSB = 8,4,2,1; oben→unten) ---
            static bool[,] EncodeHhMmSsTo6x4(TimeParts t)
            {
                int[] digits = {
                t.Hour / 10, t.Hour % 10,
                t.Minute / 10, t.Minute % 10,
                t.Second / 10, t.Second % 10
            };

                var grid = new bool[6, 4]; // [col,row]
                for (int c = 0; c < 6; c++)
                {
                    bool[] bits = EncodeDigitBCD(digits[c]);    // 4 Bits
                    for (int r = 0; r < 4; r++) grid[c, r] = bits[r];
                }
                return grid;
            }

            // --- Ziffer 0..9 → 4 Bits (8,4,2,1) ---
            static bool[] EncodeDigitBCD(int d)
            {
                if (d < 0 || d > 9) throw new ArgumentOutOfRangeException("digit");
                return new bool[] { (d & 8) != 0, (d & 4) != 0, (d & 2) != 0, (d & 1) != 0 };
            }

            // --- Binäruhr in Blockschrift (6 Spalten × 4 Reihen, jede Reihe „hochskaliert“) ---
            static void RenderBinaryBig(bool[,] grid)
            {
                var sb = new StringBuilder(512);

                for (int bitRow = 0; bitRow < 4; bitRow++)           // 4 Bitreihen (MSB→LSB)
                {
                    for (int sub = 0; sub < BIT_HEIGHT; sub++)       // vertikal strecken
                    {
                        for (int col = 0; col < 6; col++)            // 6 Spalten: H H M M S S
                        {
                            var cell = grid[col, bitRow] ? BIT1[sub] : BIT0[sub];
                            sb.Append(cell);

                            if (col < 5)
                            {
                                sb.Append(' ', BIN_COL_GAP);
                                if (col == 1 || col == 3) sb.Append(' ', BIN_GROUP_GAP);
                            }
                        }
                        sb.AppendLine();
                    }
                }
                Console.Write(sb.ToString());
            }

            // --- Große ASCII-Zeit (5 Zeilen hoch) ---
            static void RenderBigTime(string text) // z. B. "08.46.04"
            {
                var chars = text.Where(c => Glyph.ContainsKey(c)).ToArray();

                for (int row = 0; row < 5; row++)
                {
                    var line = new StringBuilder();
                    for (int i = 0; i < chars.Length; i++)
                    {
                        line.Append(Glyph[chars[i]][row]);
                        bool isSep = chars[i] == '.';
                        if (i < chars.Length - 1) line.Append(isSep ? "   " : " ");
                    }
                    Console.WriteLine(line.ToString());
                }
            }
        }
    }

```
