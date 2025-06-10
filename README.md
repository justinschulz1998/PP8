# PP8

## Goal

In this exercise you will:

* Parse **command-line flags** and **parameters** in C using `getopt` and `atoi`.
* Prompt for **interactive** input using `scanf` and demonstrate file scanning with `fscanf`.
* Read input from **standard input** via shell redirection.
* Implement and modularize a **symmetric Caesar cipher** and extend it to a **prototype asymmetric** XOR cipher, compiling and linking multiple source files.

**Important:** Start a stopwatch when you begin and work uninterruptedly for **90 minutes**. When time is up, stop immediately and record exactly where you paused.

---

## Workflow

1. **Fork** this repository on GitHub.
2. **Clone** your fork locally.
3. Create a `solutions/` directory at the project root:

   ```bash
   mkdir solutions
   ```
4. Add each task’s source file under `solutions/` (e.g., `solutions/getopt_flags.c`).
5. **Commit** and **push** your changes to GitHub.
6. **Submit** your GitHub repository link for review.

---

## Prerequisites

* GNU C compiler (`gcc`).
* Familiarity with:

  * `getopt` (include `<unistd.h>`)
  * `atoi` (include `<stdlib.h>`)
  * Standard I/O: `printf`, `scanf`, `fscanf`, `fgets`
  * Shell I/O redirection
  * Manual compilation and linking
* Consult man-pages:

  ```bash
  man getopt   # option parsing
  man atoi     # string to integer conversion
  man scanf    # interactive input
  man fscanf   # file scanning
  ```

---

## Tasks

### Task 1: Command-line Flags Only

**Objective:** Parse simple flags without arguments.

1. Create `solutions/getopt_flags.c`:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>

   int main(int argc, char *argv[]) {
       int opt;
       int flag_a = 0, flag_b = 0;

       while ((opt = getopt(argc, argv, "ab")) != -1) {
           if      (opt == 'a') flag_a = 1;
           else if (opt == 'b') flag_b = 1;
           else {
               fprintf(stderr, "Usage: %s [-a] [-b]\n", argv[0]);
               exit(EXIT_FAILURE);
           }
       }

       printf("flag_a=%d, flag_b=%d\n", flag_a, flag_b);
       return 0;
   }
   ```
2. Compile and test:

   ```bash
   gcc -o solutions/getopt_flags solutions/getopt_flags.c
   ./solutions/getopt_flags -a -b
   ```

---

### Task 2: Command-line Parameters & File Names

**Objective:** Parse options with arguments, including input and output file names.

1. Create `solutions/getopt_params.c`:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>

   int main(int argc, char *argv[]) {
       int opt;
       int number = 0;
       char *str = NULL;
       char *infile = NULL, *outfile = NULL;

       while ((opt = getopt(argc, argv, "n:s:i:o:")) != -1) {
           if      (opt == 'n') number  = atoi(optarg);
           else if (opt == 's') str     = optarg;
           else if (opt == 'i') infile  = optarg;
           else if (opt == 'o') outfile = optarg;
           else {
               fprintf(stderr, "Usage: %s -n num -s str -i infile -o outfile\n", argv[0]);
               exit(EXIT_FAILURE);
           }
       }

       printf("number=%d, string=%s, infile=%s, outfile=%s\n",
              number,
              str     ? str     : "(null)",
              infile  ? infile  : "(none)",
              outfile ? outfile : "(none)");
       return 0;
   }
   ```
2. Compile and test:

   ```bash
   gcc -o solutions/getopt_params solutions/getopt_params.c
   ./solutions/getopt_params -n 42 -s hello -i input.txt -o output.txt
   ```

#### Reflection Questions

1. **How do you pass a file name to a program using the `-i` and `-o` options?**
   Der Dateiname wird als argument nach der Flag übergeben.
3. **What are typical use cases for parameters versus flags? How do the differ from one another?**
Flags werden zur Steuerung genutzt Parameter sind meist veränderliche Variablen zur Funktiomn des Programms.
---

### Task 3: Interactive Input with `scanf` & `fscanf`

**Objective:** Prompt for user input at runtime and read structured data from a file.

1. Create `solutions/interactive.c`:

   ```c
   #include <stdio.h>
   #define BUF_SIZE 100

   int main(void) {
       char name[BUF_SIZE];
       int age;

       printf("Enter your name: ");
       scanf("%99s", name);
       printf("Hello, %s!\n", name);

       printf("Enter your age: ");
       scanf("%d", &age);
       printf("You are %d years old.\n", age);

       // File scanning
       FILE *fp = fopen("data.txt", "r");
       if (!fp) { perror("fopen"); return 1; }
       char fname[BUF_SIZE];
       int fage;
       if (fscanf(fp, "%99s %d", fname, &fage) == 2) {
           printf("File data: %s is %d years old.\n", fname, fage);
       }
       fclose(fp);
       return 0;
   }
   ```
2. Provide `solutions/data.txt` containing:

   ```text
   Alice 30
   ```
3. Compile and run:

   ```bash
   gcc -o solutions/interactive solutions/interactive.c
   ./solutions/interactive
   ```

#### Reflection Question

* **Why is a run-to-completion (batch) approach often preferable to interactive input?**

Automatisierbar , Vermeidung von Wartezeiten und Wiederholbarkeit. 
---

### Task 4: Input Redirection from STDIN

**Objective:** Read input supplied via shell redirection.

1. Create `solutions/redirect_input.c`:

   ```c
   #include <stdio.h>
   #define BUF_SIZE 256

   int main(void) {
       char buf[BUF_SIZE];
       if (fgets(buf, BUF_SIZE, stdin))
           printf("You entered: %s", buf);
       return 0;
   }
   ```
2. Create `solutions/input.txt` with sample text.
3. Compile and run:

   ```bash
   gcc -o solutions/redirect_input solutions/redirect_input.c
   ./solutions/redirect_input < solutions/input.txt
   ```

#### Reflection Question

* **What is the difference between redirecting to stdin and explicitly opening a file with `fopen`?**
Bei fopen wird direkt auf die Datei zugegriffen. Bei stdin wird diese lediglich als Eingabequelle genutzt. 
---

### Task 5: Caesar Cipher & Prototype Asymmetric XOR Cipher

**Objective:** Implement and modularize two cipher algorithms.

#### 5.1 Symmetric Caesar Cipher

1. **Header (`cipher.h`)**:

   ```c
   #ifndef CIPHER_H
   #define CIPHER_H
   char encrypt_char(char c, int shift);
   char decrypt_char(char c, int shift);
   #endif
   ```
2. **Implementation (`cipher.c`)**:

   ```c
   #include "cipher.h"

   char encrypt_char(char c, int shift) {
       if (c >= 'A' && c <= 'Z') return 'A' + (c - 'A' + shift) % 26;
       if (c >= 'a' && c <= 'z') return 'a' + (c - 'a' + shift) % 26;
       return c;
   }

   char decrypt_char(char c, int shift) {
       return encrypt_char(c, 26 - (shift % 26));
   }
   ```
3. **Driver (`caesar.c`)**:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include "cipher.h"

   void usage(const char *prog) {
       fprintf(stderr, "Usage: %s -e shift|-d shift -i infile -o outfile\n", prog);
       exit(EXIT_FAILURE);
   }

   int main(int argc, char *argv[]) {
       int opt, shift = 0, enc = -1;
       char *infile = NULL, *outfile = NULL;

       while ((opt = getopt(argc, argv, "e:d:i:o:")) != -1) {
           if      (opt == 'e') { shift = atoi(optarg); enc = 1; }
           else if (opt == 'd') { shift = atoi(optarg); enc = 0; }
           else if (opt == 'i') infile = optarg;
           else if (opt == 'o') outfile = optarg;
           else usage(argv[0]);
       }

       if (enc < 0 || !infile || !outfile) usage(argv[0]);

       FILE *fin = fopen(infile, "r");
       if (!fin) { perror("fopen infile"); exit(EXIT_FAILURE); }
       FILE *fout = fopen(outfile, "w");
       if (!fout) { perror("fopen outfile"); exit(EXIT_FAILURE); }

       int c;
       while ((c = fgetc(fin)) != EOF) {
           char out;
           if (enc)
               out = encrypt_char(c, shift);
           else
               out = decrypt_char(c, shift);
           fputc(out, fout);
       }

       fclose(fin);
       fclose(fout);
       return 0;
   }
   ```
4. **Compile & link**:

   ```bash
   gcc -c solutions/cipher.c -o solutions/cipher.o
   gcc -c solutions/caesar.c -o solutions/caesar.o
   gcc solutions/caesar.o solutions/cipher.o -o solutions/caesar
   ```
5. **Test**:

   ```bash
   ./solutions/caesar -e 3 -i input.txt -o enc.txt
   ./solutions/caesar -d 3 -i enc.txt -o dec.txt
   ```

#### 5.2 Prototype Asymmetric XOR Cipher

1. **Header (`asym.h`)**:

   ```c
   #ifndef ASYM_H
   #define ASYM_H
   char encrypt_xor(char c, char key);
   char decrypt_xor(char c, char key);
   #endif
   ```
2. **Implementation (`asym.c`)**:

   ```c
   #include "asym.h"

   char encrypt_xor(char c, char key) { return c ^ key; }
   char decrypt_xor(char c, char key) { return c ^ key; }
   ```
3. **Driver (`advanced_cipher.c`)**:

   ```c
   #include <stdio.h>
   #include <stdlib.h>
   #include <string.h>
   #include <unistd.h>
   #include "cipher.h"
   #include "asym.h"

   void usage(const char *prog) {
       fprintf(stderr, "Usage: %s -m <caesar|xor> -e key|-d key -i infile -o outfile\n", prog);
       exit(EXIT_FAILURE);
   }

   int main(int argc, char *argv[]) {
       int opt, enc = -1, key = 0;
       char *mode = NULL, *infile = NULL, *outfile = NULL;

       while ((opt = getopt(argc, argv, "m:e:d:i:o:")) != -1) {
           if      (opt == 'm') mode = optarg;
           else if (opt == 'e') { key = atoi(optarg); enc = 1; }
           else if (opt == 'd') { key = atoi(optarg); enc = 0; }
           else if (opt == 'i') infile = optarg;
           else if (opt == 'o') outfile = optarg;
           else usage(argv[0]);
       }

       if (!mode || enc < 0 || !infile || !outfile) usage(argv[0]);

       FILE *fin = fopen(infile, "r");
       if (!fin) { perror("fopen infile"); exit(EXIT_FAILURE); }
       FILE *fout = fopen(outfile, "w");
       if (!fout) { perror("fopen outfile"); exit(EXIT_FAILURE); }

       int c;
       while ((c = fgetc(fin)) != EOF) {
           char out;
           if (strcmp(mode, "caesar") == 0) {
               if (enc)
                   out = encrypt_char(c, key);
               else
                   out = decrypt_char(c, key);
           } else if (strcmp(mode, "xor") == 0) {
               if (enc)
                   out = encrypt_xor(c, (char)key);
               else
                   out = decrypt_xor(c, (char)key);
           } else {
               usage(argv[0]);
           }
           fputc(out, fout);
       }

       fclose(fin);
       fclose(fout);
       return 0;
   }
   ```
4. **Compile & link**:

   ```bash
   gcc -c solutions/cipher.c solutions/asym.c solutions/advanced_cipher.c
   gcc solutions/cipher.o solutions/asym.o solutions/advanced_cipher.o -o solutions/advanced_cipher
   ```
5. **Test both modes**:

   ```bash
   ./solutions/advanced_cipher -m caesar -e 3 -i input.txt -o enc.txt
   ./solutions/advanced_cipher -m caesar -d 3 -i enc.txt -o dec.txt
   ./solutions/advanced_cipher -m xor    -e 42 -i input.txt -o xor_enc.txt
   ./solutions/advanced_cipher -m xor    -d 42 -i xor_enc.txt -o xor_dec.txt
   ```

#### Reflection Question

* **Explain in your own words what the encryption and decryption processes are doing in both ciphers.**

---

**Remember:** Stop after **90 minutes** and record where you stopped.
