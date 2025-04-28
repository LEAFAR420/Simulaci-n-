# Simulacion

import java.util.ArrayList;
import java.util.HashSet;
import java.util.List;
import java.util.Scanner;
import java.util.Set;

/**
 * Implementa un Generador Congruencial Lineal Mixto (GCLM).
 * La fórmula utilizada es: Xn+1 = (a * Xn + c) mod m
 *
 * Esta versión solicita los parámetros a, c, Xo y m al usuario.
 * Verifica si los parámetros ingresados cumplen las condiciones del
 * Teorema de Hull-Dobell para garantizar un período completo (longitud m).
 * Si no cumplen, notifica al usuario y solicita nuevos parámetros.
 * Finalmente, genera una secuencia de números y explica si el período
 * completo está garantizado teóricamente y por qué.
 *
 * @author Hernandez Villa Emilio, Rico Gonzalez Henry Rafael
 */
public class GeneradorMixtoV2 {

    // --- Parámetros del generador ---
    private long a; // Multiplicador
    private long c; // Incremento (constante aditiva)
    private long m; // Módulo
    private long semillaActual; // Xn (el estado actual del generador)

    /**
     * Constructor para el Generador Congruencial Lineal Mixto.
     * Inicializa el generador con los parámetros dados y la semilla inicial.
     * La validación de los parámetros para período completo debe hacerse *antes*
     * de llamar a este constructor usando los métodos estáticos de validación.
     *
     * @param a Multiplicador (0 <= a < m).
     * @param c Incremento (0 <= c < m).
     * @param m Módulo (m > 0).
     * @param Xo Semilla inicial (0 <= Xo < m).
     * @throws IllegalArgumentException si m <= 0, o si a, c, Xo están fuera de rango.
     */
    public GeneradorMixtoV2(long a, long c, long m, long Xo) {
        if (m <= 0) {
            throw new IllegalArgumentException("El módulo 'm' debe ser positivo.");
        }
        if (a < 0 || a >= m) {
            throw new IllegalArgumentException("El multiplicador 'a' debe estar entre 0 y m-1.");
        }
        if (c < 0 || c >= m) {
            throw new IllegalArgumentException("El incremento 'c' debe estar entre 0 y m-1.");
        }
        if (Xo < 0 || Xo >= m) {
            throw new IllegalArgumentException("La semilla inicial 'Xo' debe estar entre 0 y m-1.");
        }

        this.a = a;
        this.c = c;
        this.m = m;
        this.semillaActual = Xo;
    }

    /**
     * Genera el siguiente número pseudoaleatorio entero en la secuencia.
     * Aplica la fórmula Xn+1 = (a * Xn + c) mod m y actualiza el estado interno.
     *
     * @return El siguiente número entero pseudoaleatorio en el rango [0, m-1].
     */
    public long siguienteEntero() {
        semillaActual = (a * semillaActual + c) % m;
        // Asegurar resultado no negativo (aunque teóricamente no debería serlo con params >= 0)
        if (semillaActual < 0) {
            semillaActual += m;
        }
        return semillaActual;
    }

    /**
     * Genera el siguiente número pseudoaleatorio normalizado en el rango [0, 1).
     *
     * @return El siguiente número flotante (double) pseudoaleatorio en el rango [0, 1).
     */
    public double siguienteNormalizado() {
        return (double) siguienteEntero() / m;
    }

    /** Obtiene la semilla actual. */
    public long getSemillaActual() {
        return semillaActual;
    }

    /** Obtiene el módulo 'm'. */
    public long getModulo() {
        return m;
    }

    /** Obtiene el multiplicador 'a'. */
    public long getMultiplicador() {
        return a;
    }

    /** Obtiene el incremento 'c'. */
    public long getIncremento() {
        return c;
    }

    // --- Métodos Estáticos de Validación (Teorema de Hull-Dobell) ---

    /**
     * Calcula el Máximo Común Divisor (GCD) de dos números usando el algoritmo de Euclides.
     *
     * @param n1 Primer número (no negativo).
     * @param n2 Segundo número (no negativo).
     * @return El GCD de n1 y n2. Devuelve n1 si n2 es 0, n2 si n1 es 0.
     */
    public static long gcd(long n1, long n2) {
         // Asegurarse de que son no negativos para el algoritmo estándar
        n1 = Math.abs(n1);
        n2 = Math.abs(n2);
        while (n2 != 0) {
            long temp = n2;
            n2 = n1 % n2;
            n1 = temp;
        }
        return n1;
    }

    /**
     * Obtiene el conjunto de factores primos únicos de un número n.
     *
     * @param n El número (mayor que 1) a factorizar.
     * @return Un Set de Long conteniendo los factores primos únicos de n.
     */
    public static Set<Long> obtenerFactoresPrimos(long n) {
        Set<Long> factores = new HashSet<>();
        if (n <= 1) return factores; // No hay factores primos para n <= 1

        // Divide por 2 tantas veces como sea posible
        if (n % 2 == 0) {
            factores.add(2L);
            while (n % 2 == 0) {
                n /= 2;
            }
        }

        // Ahora n debe ser impar. Iterar solo impares hasta sqrt(n).
        for (long i = 3; i * i <= n; i += 2) {
            if (n % i == 0) {
                factores.add(i);
                while (n % i == 0) {
                    n /= i;
                }
            }
        }

        // Si n es todavía mayor que 1 después del bucle, n es primo.
        if (n > 1) {
            factores.add(n);
        }
        return factores;
    }

    /**
     * Verifica si los parámetros a, c, m cumplen las condiciones del Teorema de
     * Hull-Dobell para garantizar un período completo (igual a m).
     * Imprime en consola el resultado de cada condición.
     *
     * Condiciones:
     * 1. c y m son relativamente primos (gcd(c, m) = 1).
     * 2. a - 1 es divisible por todos los factores primos de m.
     * 3. Si m es divisible por 4, entonces a - 1 también debe ser divisible por 4.
     *
     * @param a El multiplicador.
     * @param c El incremento.
     * @param m El módulo.
     * @return true si todas las condiciones se cumplen, false en caso contrario.
     */
    public static boolean validarParametrosHullDobell(long a, long c, long m) {
        System.out.println("\n--- Verificando Condiciones de Hull-Dobell para Período Completo (m=" + m + ") ---");
        boolean todasLasCondicionesOk = true;

        // --- Condición 1: gcd(c, m) = 1 ---
        long gcd_c_m = gcd(c, m);
        boolean cond1Ok = (gcd_c_m == 1);
        System.out.println(" 1. ¿c (" + c + ") y m (" + m + ") son relativamente primos? (gcd=" + gcd_c_m + ")");
        System.out.println("    Resultado: " + (cond1Ok ? "Sí" : "No"));
        if (!cond1Ok) todasLasCondicionesOk = false;

        // --- Condición 2: a - 1 es divisible por todos los factores primos de m ---
        Set<Long> factoresPrimosM = obtenerFactoresPrimos(m);
        boolean cond2Ok = true;
        System.out.println(" 2. ¿a-1 (" + (a - 1) + ") es divisible por todos los factores primos de m " + factoresPrimosM + "?");
        if (factoresPrimosM.isEmpty() && m > 1) { // Caso m>1 sin factores? Imposible teóricamente, pero seguro
             System.out.println("    Error: No se pudieron encontrar factores primos para m=" + m);
             cond2Ok = false;
        } else if (factoresPrimosM.isEmpty() && m <= 1) {
             System.out.println("    Condición no aplicable (m <= 1)");
             // Se considera OK si m=1, aunque el generador es trivial
        }
        else {
            for (long p : factoresPrimosM) {
                if ((a - 1) % p != 0) {
                    System.out.println("    Fallo: a-1 (" + (a - 1) + ") NO es divisible por el factor primo " + p);
                    cond2Ok = false;
                    break; // Basta un fallo
                }
            }
        }
        System.out.println("    Resultado: " + (cond2Ok ? "Sí" : "No"));
        if (!cond2Ok) todasLasCondicionesOk = false;


        // --- Condición 3: Si m es divisible por 4, entonces a - 1 es divisible por 4 ---
        boolean cond3Ok = true; // Asumir verdad si la premisa (m % 4 == 0) es falsa
        System.out.println(" 3. ¿Si m ("+m+") es divisible por 4, entonces a-1 (" + (a - 1) + ") es divisible por 4?");
        if (m % 4 == 0) {
            System.out.println("    Premisa: m ("+m+") es divisible por 4.");
            if ((a - 1) % 4 != 0) {
                System.out.println("    Fallo: a-1 (" + (a - 1) + ") NO es divisible por 4.");
                cond3Ok = false;
            } else {
                 System.out.println("    Consecuencia: a-1 (" + (a - 1) + ") SÍ es divisible por 4.");
            }
        } else {
             System.out.println("    Premisa: m ("+m+") NO es divisible por 4. La condición se cumple vacuamente.");
        }
        System.out.println("    Resultado: " + (cond3Ok ? "Sí" : "No"));
        if (!cond3Ok) todasLasCondicionesOk = false;

        System.out.println("------------------------------------------------------------------");

        return todasLasCondicionesOk;
    }

    // --- Método Principal (main) ---

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        long m = 0, a = 0, c = 0, Xo = 0;
        boolean parametrosValidos = false;

        System.out.println("=== Generador Congruencial Lineal Mixto (Validación de Período Completo) ===");

        do {
            // --- Solicitar Parámetros ---
            System.out.println("\nIngrese los parámetros para el generador (Xn+1 = (a*Xn + c) mod m):");

            // Solicitar m
            boolean mOk = false;
            while (!mOk) {
                System.out.print(" - Módulo m (entero > 0): ");
                try {
                    m = scanner.nextLong();
                    if (m > 0) {
                        mOk = true;
                    } else {
                        System.out.println("   Error: m debe ser mayor que 0.");
                    }
                } catch (Exception e) {
                    System.out.println("   Error: Ingrese un número entero válido.");
                    scanner.next(); // Limpiar buffer
                }
            }

            // Solicitar a
            boolean aOk = false;
            while (!aOk) {
                System.out.print(" - Multiplicador a (0 <= a < " + m + "): ");
                try {
                    a = scanner.nextLong();
                    if (a >= 0 && a < m) {
                        aOk = true;
                    } else {
                        System.out.println("   Error: a debe estar en el rango [0, m-1].");
                    }
                } catch (Exception e) {
                    System.out.println("   Error: Ingrese un número entero válido.");
                    scanner.next(); // Limpiar buffer
                }
            }

            // Solicitar c
            boolean cOk = false;
            while (!cOk) {
                 // Para período completo, c debe ser > 0 teóricamente. Aunque GCL Mixto permite c=0.
                 // Lo validaremos en Hull-Dobell. Permitimos c=0 aquí.
                System.out.print(" - Incremento c (0 <= c < " + m + "): ");
                try {
                    c = scanner.nextLong();
                     if (c >= 0 && c < m) {
                        cOk = true;
                    } else {
                        System.out.println("   Error: c debe estar en el rango [0, m-1].");
                    }
                } catch (Exception e) {
                    System.out.println("   Error: Ingrese un número entero válido.");
                    scanner.next(); // Limpiar buffer
                }
            }

            // Solicitar Xo
            boolean xoOk = false;
            while (!xoOk) {
                System.out.print(" - Semilla Xo (0 <= Xo < " + m + "): ");
                try {
                    Xo = scanner.nextLong();
                    if (Xo >= 0 && Xo < m) {
                        xoOk = true;
                    } else {
                        System.out.println("   Error: Xo debe estar en el rango [0, m-1].");
                    }
                } catch (Exception e) {
                    System.out.println("   Error: Ingrese un número entero válido.");
                    scanner.next(); // Limpiar buffer
                }
            }

            // --- Validar Parámetros para Período Completo ---
            parametrosValidos = validarParametrosHullDobell(a, c, m);

            if (!parametrosValidos) {
                System.out.println("\n*** Los parámetros ingresados NO garantizan un período completo según el Teorema de Hull-Dobell. ***");
                System.out.println("Por favor, ingrese un nuevo conjunto de parámetros.");
            }

        } while (!parametrosValidos); // Repetir hasta que los parámetros sean válidos para período completo

        // --- Parámetros Válidos - Crear Generador y Mostrar Resultados ---
        System.out.println("\n¡Éxito! Los parámetros ingresados cumplen las condiciones para un período completo (m=" + m + ").");

        GeneradorMixtoV2 generador = new GeneradorMixtoV2(a, c, m, Xo);

        System.out.println("\n--- Generador Inicializado ---");
        System.out.println(" Parámetros:");
        System.out.println("   a (multiplicador) = " + generador.getMultiplicador());
        System.out.println("   c (incremento)    = " + generador.getIncremento());
        System.out.println("   m (módulo)        = " + generador.getModulo());
        System.out.println("   Xo (semilla)      = " + generador.getSemillaActual()); // Muestra la Xo inicial
        System.out.println("-----------------------------");

        // Generar y mostrar los primeros N números (o hasta m si m es pequeño)
        //int N = 20; // Límite de números a mostrar
        //long limite = Math.min(N, m); // Mostrar máximo N o m números
        long limite = m;

        System.out.println("\n--- Primeros " + limite + " números generados ---");
        System.out.println(" i | Entero (Xi) | Normalizado (Ui)");
        System.out.println("---|-------------|------------------");
        System.out.printf("%2d | %11d | %s%n", 0, generador.getSemillaActual(), "(Semilla Xo)");

        for (int i = 1; i <= limite; i++) {
            long entero = generador.siguienteEntero();
            double normalizado = (double) entero / generador.getModulo();
            System.out.printf("%2d | %11d | %.8f%n", i, entero, normalizado);
             // Si generamos m números, el siguiente debería ser X1 de nuevo
            if (i == m) {
                System.out.println("... (Se han generado m=" + m + " números, completando el ciclo teórico)");
            }
        }
         //if (limite < m && limite == m) {
           //  System.out.println("... (mostrando solo los primeros " + m + ")");
         //}
        System.out.println("--------------------------------------");

        // --- Conclusión sobre el Período ---
        System.out.println("\n--- Conclusión sobre el Período ---");
        System.out.println("Basado en la verificación del Teorema de Hull-Dobell:");
        System.out.println("  - Condición 1 (gcd(c, m) = 1): Cumplida.");
        System.out.println("  - Condición 2 (a-1 divisible por factores primos de m): Cumplida.");
        System.out.println("  - Condición 3 (Si m%4=0 => (a-1)%4=0): Cumplida.");
        System.out.println("\n>> Este generador TIENE GARANTIZADO un período completo de longitud m = " + m + ".");
        System.out.println("   Esto significa que generará todos los números enteros de 0 a " + (m-1) + " exactamente una vez");
        System.out.println("   antes de repetir la secuencia, independientemente de la semilla inicial Xo.");
        System.out.println("============================================================================");

        scanner.close(); // Cerrar scanner al final
    }
}
