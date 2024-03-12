package com.akshatsoni;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.*;

class CustomPrint {
    /**
     * get - shouldPrintComments
     */
    static final boolean shouldPrintComments = true;

    /**
     * @param obj Object to print on new line
     */
    static void println(Object obj) {
        if (!shouldPrintComments)
            return;
        System.out.println(obj);
    }

    /**
     * @param TAG Tag to the print
     * @param obj Object to print on new line
     */
    static void println(String TAG, Object obj) {
        if (!shouldPrintComments)
            return;
        System.out.println("[" + TAG + "] :: " + obj);
    }

    /**
     * @param obj Object to print error on new line
     */
    static void printError(Object obj) {
        if (!shouldPrintComments)
            return;
        System.err.println(obj);
    }

    /**
     * @param TAG Tag to the print
     * @param obj Object to print error on new line
     */
    static void printError(String TAG, Object obj) {
        if (!shouldPrintComments)
            return;
        System.err.println("[" + TAG + "] :: " + obj);
    }
}

class ReadFile {
    /**
     * @param pathToFile the path of the file to read
     * @return set of words in the file
     * @author Akshat Soni
     */
    public static Set<String> readWordsFromFile(String pathToFile) {
        // If filepath is null or empty return the function
        if (pathToFile == null || pathToFile.isEmpty()) {
            return null;
        }

        // Create a set of string to store each word in the file
        Set<String> wordsInFile = new TreeSet<>();

        // try to read the file and populate each word in the set of string
        try (BufferedReader bufferedReader = new BufferedReader(new FileReader(pathToFile))) {
            String lineInFile;
            while ((lineInFile = bufferedReader.readLine()) != null) {
                // Feed line after making it case in-sensitive
                wordsInFile.addAll(extractWordsFromString(lineInFile.toLowerCase()));
            }
        } catch (IOException e) {
            // Print the exception stack trace if there is any issue in reading the file
            CustomPrint.printError("Error", e.getMessage());
        }

        return wordsInFile;
    }

    /**
     * @param line a line to separate the words in it
     * @return set of separated words
     * @author Akshat Soni
     */
    private static Set<String> extractWordsFromString(String line) {
        // If line is null or empty return the function
        if (line == null || line.isEmpty()) {
            return null;
        }

        // set of string to store the words in this line
        Set<String> words = new TreeSet<>();

        // Split the words by whitespace
        String[] split = line.split("\\s+");

        for (String word : split) {
            // Add words of line and clean it
            words.add(word.replaceAll("[^a-zA-Z]", ""));
        }

        return words;
    }
}

class LongestCommonSubsequence {
    /**
     * Return the lcs length of two words.
     *
     * @param cmpWord1 Word1 to check for LCS
     * @param cmpWord2 Word2 to check for LCS
     * @return LCS value
     */
    public static int LCS(String cmpWord1, String cmpWord2) {
        // if either of the compare word is null or empty then return the function
        if (cmpWord1 == null || cmpWord1.isEmpty() || cmpWord2 == null || cmpWord2.isEmpty()) {
            return -1;
        }

        int[][] lengthsOfWords = new int[cmpWord1.length() + 1][cmpWord2.length() + 1];

        for (int lenWord1 = 0; lenWord1 < cmpWord1.length(); lenWord1++) {
            for (int lenWord2 = 0; lenWord2 < cmpWord2.length(); lenWord2++) {
                if (cmpWord1.charAt(lenWord1) == cmpWord2.charAt(lenWord2)) {
                    lengthsOfWords[lenWord1 + 1][lenWord2 + 1] = lengthsOfWords[lenWord1][lenWord2] + 1;
                } else {
                    lengthsOfWords[lenWord1 + 1][lenWord2 + 1] = Math.max(lengthsOfWords[lenWord1 + 1][lenWord2],
                            lengthsOfWords[lenWord1][lenWord2 + 1]);
                }
            }
        }

        return lengthsOfWords[cmpWord1.length()][cmpWord2.length()];
    }

    /**
     * function to get the last key of provided hashmap
     *
     * @param lcsDictionary HashMap of which the key is to be found
     */
    static int getLastKey(HashMap<Integer, List<String>> lcsDictionary) {
        // if hashmap is null or empty return invalid last key
        if (lcsDictionary == null || lcsDictionary.isEmpty()) {
            return -1;
        }

        List<Map.Entry<Integer, List<String>>> entries = new ArrayList<>(lcsDictionary.entrySet());
        int lastKey = -1;

        if (!entries.isEmpty()) {
            Map.Entry<Integer, List<String>> lastEntry = entries.get(entries.size() - 1);
            lastKey = lastEntry.getKey();
        } else {
            CustomPrint.printError("LastKey","The map is empty");
        }

        return lastKey;
    }

    /**
     * Provides the list of words that are near to the input word
     *
     * @param misspelled The input word to check with other words
     */
    public static List<String> suggestCorrection(String misspelled) {
        HashMap<Integer, List<String>> lcsDictionary = new HashMap<>();
        int threshold = 1;

        for (String word : SpellChecker.bagOfWords) {
            int lcsValue = LCS(misspelled, word);

            if (lcsValue < 0) {
                continue;
            }

            if (lcsDictionary.containsKey(lcsValue)) {
                lcsDictionary.get(lcsValue).add(word);
            } else {
                lcsDictionary.put(lcsValue, new ArrayList<>());
                lcsDictionary.get(lcsValue).add(word);
            }
        }

        int lastKey = getLastKey(lcsDictionary);
        if (lastKey < 0) {
            CustomPrint.println("ERROR", "Unable to find last key");
        }

        List<String> maxLcsWords = new ArrayList<>();
        for (String words : lcsDictionary.get(lastKey)) {
            if (words.length() <= misspelled.length() + threshold) {
                maxLcsWords.add(words);
            }
        }

        if (!maxLcsWords.isEmpty()) {
            return maxLcsWords;
        } else {
            return null;
        }
    }
}

public class SpellChecker {
    public static Set<String> bagOfWords = new HashSet<>();

    public static void main(String[] args) {
        Scanner inputScanner = new Scanner(System.in);
        String shouldContinue = "y";

        CustomPrint.println("Instruction", "This program will check for the input word's spelling. " +
                "If it is incorrect it will provide some suggestions.");
        CustomPrint.println("Progress", "Reading the file for populating the words to match with...");
        String pathToFile = ".\\src\\doc\\non-existing.txt";

        bagOfWords = ReadFile.readWordsFromFile(pathToFile);

        if (bagOfWords == null || bagOfWords.isEmpty()) {
            CustomPrint.println("ERROR", "Unable to read the words from the file.");
        }

        while (shouldContinue.equalsIgnoreCase("y")) {
            CustomPrint.println("Word", "Enter your word to check the spelling: ");
            String inputWord = inputScanner.next();

            if (inputWord == null || inputWord.isEmpty()) {
                return;
            }
            if (bagOfWords.contains(inputWord)) {
                CustomPrint.println("Result", "The input word is correct.");
            } else {
                List<String> suggestedWords = LongestCommonSubsequence.suggestCorrection(inputWord);
                if (suggestedWords == null || suggestedWords.isEmpty()) {
                    CustomPrint.println("Result", "Sorry! No suggestions found");
                } else {
                    CustomPrint.println("Result", suggestedWords);
                }
            }

            CustomPrint.println("Do you want to continue (y/n): ");
            shouldContinue = inputScanner.next();
        }

        CustomPrint.println("Thanks for using the program!");
        inputScanner.close();
    }
}
