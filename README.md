# AI


1. Program to compute FIRST and FOLLOW - 

import java.util.*;

public class FirstFollow {
    static Map<String, List<String>> grammar = new HashMap<>();
    static Map<String, Set<String>> first = new HashMap<>();
    static Map<String, Set<String>> follow = new HashMap<>();
    static String startSymbol;

    public static void main(String[] args) {
        // Grammar setup
        addProduction("E", "T E'");
        addProduction("E'", "+ T E'");
        addProduction("E'", "ε");
        addProduction("T", "F T'");
        addProduction("T'", "* F T'");
        addProduction("T'", "ε");
        addProduction("F", "( E )");
        addProduction("F", "id");

        startSymbol = "E"; // start symbol

        // Compute FIRST sets
        for (String nonTerminal : grammar.keySet()) {
            first.put(nonTerminal, computeFirst(nonTerminal));
        }

        // Initialize FOLLOW sets
        for (String nonTerminal : grammar.keySet()) {
            follow.put(nonTerminal, new HashSet<>());
        }
        follow.get(startSymbol).add("$"); // End marker

        // Compute FOLLOW sets
        boolean changed;
        do {
            changed = false;
            for (String lhs : grammar.keySet()) {
                for (String production : grammar.get(lhs)) {
                    String[] symbols = production.split(" ");
                    for (int i = 0; i < symbols.length; i++) {
                        String B = symbols[i];
                        if (!grammar.containsKey(B)) continue; // skip terminals

                        Set<String> followB = follow.get(B);
                        int prevSize = followB.size();

                        // Get FIRST of the rest
                        Set<String> firstOfRest = new HashSet<>();
                        boolean nullable = true;
                        for (int j = i + 1; j < symbols.length; j++) {
                            String next = symbols[j];
                            Set<String> firstNext = grammar.containsKey(next)
                                    ? new HashSet<>(first.get(next))
                                    : new HashSet<>(Collections.singleton(next));
                            firstOfRest.addAll(firstNext);
                            if (!firstNext.contains("ε")) {
                                nullable = false;
                                break;
                            }
                        }

                        firstOfRest.remove("ε");
                        followB.addAll(firstOfRest);

                        if (nullable || i + 1 == symbols.length) {
                            followB.addAll(follow.get(lhs));
                        }

                        if (followB.size() != prevSize) changed = true;
                    }
                }
            }
        } while (changed);

        // Print FIRST
        System.out.println("FIRST Sets:");
        for (String nt : first.keySet()) {
            System.out.println("FIRST(" + nt + ") = " + first.get(nt));
        }

        // Print FOLLOW
        System.out.println("\nFOLLOW Sets:");
        for (String nt : follow.keySet()) {
            System.out.println("FOLLOW(" + nt + ") = " + follow.get(nt));
        }
    }

    static void addProduction(String lhs, String rhs) {
        grammar.computeIfAbsent(lhs, k -> new ArrayList<>()).add(rhs);
    }

    static Set<String> computeFirst(String symbol) {
        Set<String> result = new HashSet<>();
        for (String production : grammar.get(symbol)) {
            String[] symbols = production.split(" ");
            for (String s : symbols) {
                if (!grammar.containsKey(s)) { // Terminal
                    result.add(s);
                    break;
                }
                Set<String> firstS = computeFirst(s);
                result.addAll(firstS);
                if (!firstS.contains("ε")) break;
            }
        }
        return result;
    }
}

2. WAP to define a MACRO without argument, expand MACRO calls and
generate expanded source code.


import java.util.*;

public class MacroProcessor {
    public static void main(String[] args) {
        List<String> inputCode = Arrays.asList(
            "MACRO",
            "INCR",
            "    MOVER AREG, BETA",
            "    ADD AREG, ONE",
            "    MOVEM AREG, ALPHA",
            "MEND",
            "START",
            "INCR",
            "END"
        );

        Map<String, List<String>> macroTable = new HashMap<>();
        List<String> expandedCode = new ArrayList<>();

        boolean inMacroDef = false;
        String currentMacroName = null;

        for (String line : inputCode) {
            line = line.trim();

            if (line.equals("MACRO")) {
                inMacroDef = true;
                continue;
            }

            if (inMacroDef && currentMacroName == null) {
                currentMacroName = line;
                macroTable.put(currentMacroName, new ArrayList<>());
                continue;
            }

            if (line.equals("MEND")) {
                inMacroDef = false;
                currentMacroName = null;
                continue;
            }

            if (inMacroDef) {
                macroTable.get(currentMacroName).add(line);
            } else if (macroTable.containsKey(line)) {
                expandedCode.addAll(macroTable.get(line));
            } else {
                expandedCode.add(line);
            }
        }

        // 🖨️ Print expanded code
        System.out.println("Expanded Source Code:\n");
        for (String line : expandedCode) {
            System.out.println(line);
        }
    }
}

3. WAP to define a MACRO with one argument, expand MACRO calls and
generate expanded source code.

import java.util.*;

public class MacroWithArg {
    static class Macro {
        String name;
        String arg;
        List<String> body = new ArrayList<>();
    }

    public static void main(String[] args) {
        List<String> input = Arrays.asList(
            "MACRO",
            "PRINT A",
            "    MOV AREG, A",
            "    OUT",
            "MEND",
            "START",
            "PRINT NUM",
            "END"
        );

        List<String> output = new ArrayList<>();
        Map<String, Macro> macroTable = new HashMap<>();

        boolean inMacro = false;
        Macro currentMacro = null;

        for (String line : input) {
            line = line.trim();
            if (line.equals("MACRO")) {
                inMacro = true;
                continue;
            } else if (line.equals("MEND")) {
                macroTable.put(currentMacro.name, currentMacro);
                inMacro = false;
                currentMacro = null;
                continue;
            }

            if (inMacro) {
                if (currentMacro == null) {
                    String[] parts = line.split(" ");
                    currentMacro = new Macro();
                    currentMacro.name = parts[0];
                    currentMacro.arg = parts[1];
                } else {
                    currentMacro.body.add(line);
                }
            } else {
                String[] parts = line.split(" ");
                if (macroTable.containsKey(parts[0])) {
                    Macro m = macroTable.get(parts[0]);
                    String actualArg = parts[1];
                    for (String macroLine : m.body) {
                        String expanded = macroLine.replaceAll("\\b" + m.arg + "\\b", actualArg);
                        output.add(expanded);
                    }
                } else {
                    output.add(line);
                }
            }
        }

        System.out.println("Expanded Source Code:");
        for (String line : output) {
            System.out.println(line);
        }
    }
}


4. Write a program to count number of characters, words, sentences, lines,
tabs, numbers and blank spaces present in input using LEX.

%{
int charCount = 0;
int wordCount = 0;
int sentenceCount = 0;
int lineCount = 0;
int tabCount = 0;
int numberCount = 0;
int spaceCount = 0;
%}

%%
[ \t]          { charCount += yyleng;
                 if (yytext[0] == ' ') spaceCount++;
                 else if (yytext[0] == '\t') tabCount++; }

[0-9]+         { numberCount++; charCount += yyleng; }

[A-Za-z]+      { wordCount++; charCount += yyleng; }

[.!?]          { sentenceCount++; charCount += yyleng; }

\n             { lineCount++; charCount++; }

.              { charCount++; }

%%

int main() {
    printf("Enter text (Ctrl+D to end):\n");
    yylex();
    printf("\n--- Count Report ---\n");
    printf("Characters: %d\n", charCount);
    printf("Words     : %d\n", wordCount);
    printf("Sentences : %d\n", sentenceCount);
    printf("Lines     : %d\n", lineCount);
    printf("Tabs      : %d\n", tabCount);
    printf("Numbers   : %d\n", numberCount);
    printf("Spaces    : %d\n", spaceCount);
    return 0;
}

int yywrap() {
    return 1;
}

Run command - 
lex count.l
gcc lex.yy.c -o count
./count

5. Write a program to recognize valid arithmetic expression that uses
operators + , - , * and / using YACC.

%{
#include <stdio.h>
#include <stdlib.h>

void yyerror(const char *s);
int yylex();
%}

%token NUMBER

%%
stmt: expr { printf("✅ Valid expression\n"); }
    ;

expr: expr '+' expr
    | expr '-' expr
    | expr '*' expr
    | expr '/' expr
    | '(' expr ')'
    | NUMBER
    ;

%%

void yyerror(const char *s) {
    printf("❌ Invalid expression\n");
}

int main() {
    printf("Enter an expression: ");
    yyparse();
    return 0;
}


command - yacc -d expr.y
lex expr.l
gcc y.tab.c lex.yy.c -o expr
./expr

5. Implement the Pass 1 for 2 pass assembler -

 def pass_one(assembly_code):
    symbol_table = {}
    location_counter = 0
    output = []

    for line in assembly_code:
        parts = line.strip().split()
        if parts[0] == "START":
            location_counter = int(parts[1])
            output.append((location_counter, line))
            continue

        label = None
        if len(parts) == 3:
            label = parts[0]
            opcode = parts[1]
            operand = parts[2]
        elif len(parts) == 2:
            opcode = parts[0]
            operand = parts[1]
        else:
            opcode = parts[0]
            operand = None

        if label:
            symbol_table[label] = location_counter

        output.append((location_counter, line))

        if opcode in ["DS", "DC"]:
            location_counter += 1
        else:
            location_counter += 1

    return symbol_table, output

# Test
code = [
    "START 100",
    "LOOP MOVER AREG, BETA",
    "     ADD AREG, ONE",
    "     MOVEM AREG, ALPHA",
    "ALPHA DS 1",
    "ONE   DC 1",
    "BETA  DC 2",
    "END"
]

symtab, processed_code = pass_one(code)

print("Symbol Table:")
for sym, addr in symtab.items():
    print(f"{sym} -> {addr}")

print("\nProcessed Code (Pass 1 Output):")
for addr, line in processed_code:
    print(f"{addr} : {line}")


6. implement the pass 2 of two pass assembler -
   import java.util.*;

public class Pass2Assembler {

    static class Symbol {
        int index;
        String symbol;
        int address;

        Symbol(int index, String symbol, int address) {
            this.index = index;
            this.symbol = symbol;
            this.address = address;
        }
    }

    static Map<Integer, Symbol> symbolTable = new HashMap<>();
    static List<String> intermediateCode = new ArrayList<>();

    public static void main(String[] args) {
        // 🧠 Sample Symbol Table
        symbolTable.put(1, new Symbol(1, "ALPHA", 205));
        symbolTable.put(2, new Symbol(2, "BETA", 206));

        // 💾 Intermediate Code from Pass 1
        intermediateCode = Arrays.asList(
            "(AD,01) (C,100)",
            "(IS,04) (RG,01) (S,2)",
            "(IS,05) (RG,02) (L,1)",
            "(DL,01) (C,1)",
            "(DL,02) (C,1)"
        );

        System.out.println("Generated Machine Code:");
        for (String line : intermediateCode) {
            processLine(line);
        }
    }

    static void processLine(String line) {
        String[] parts = line.split(" ");
        if (line.contains("AD")) {
            // Assembler Directive — ignore in pass 2
            return;
        } else if (line.contains("DL")) {
            // Data Definition — use constant
            if (line.contains("DL,01")) { // DC
                String value = parts[1].replaceAll("[^0-9]", "");
                System.out.printf("000\t+%s\n", value);
            } else if (line.contains("DL,02")) { // DS
                System.out.println("000\t---"); // Placeholder for reserved space
            }
        } else if (line.contains("IS")) {
            String opcode = parts[0].replaceAll("[^0-9]", "");
            String reg = parts[1].replaceAll("[^0-9]", "");
            String operand = parts[2].contains("S")
                ? String.valueOf(symbolTable.get(Integer.parseInt(parts[2].replaceAll("[^0-9]", ""))).address)
                : parts[2].replaceAll("[^0-9]", "");

            System.out.printf("%s\t%s\t%s\n", opcode, reg, operand);
        }
    }
}
 
