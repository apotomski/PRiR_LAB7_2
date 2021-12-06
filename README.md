# PRiR_LAB7_2
Zadanie nr 2 z laboratoriów 7 z przedmiotu PRiR. Implementacja gry w życie z wykorzystaniem mpi. Każda komurka przyjmuje 1 z 2 stanów w zależności od stanów jego sąsiadów oraz własnego stanu.
Program wykorzystuje mpi aby program działał na oddzielnych wątkach.

OPIS KODU

import niezbędnych bibliotek i zdefiniowania stałych

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include "mpi.h"
    #define LICZBA_ITERACJI 32
    #define ROZMIAR 16
    #define SIATKA 256
    
 funkcja rysująca siatkę stanów komurek
    
    void rysuj(int arr[]) {
    for (j = 0; j < SIATKA; j++) {
                if (j % ROZMIAR == 0) {
                    printf("\n");
                }
                printf("%d  ", arr[j]);
            }
            printf("\n");
    }
    
  funkcja modulo  
    
     int MOD(int a, int b) {
    if(a % b < 0)
        return a % b + b;
    else
        return a % b;
    }
   
MAIN
 sprawdzenie liczby argumentów i przypisanie odpowiedniej wartości dla si lub zakończenie programu
 
     if (argc == 1) 
            si = LICZBA_ITERACJI;
        else if (argc == 2) 
            si = atoi(argv[1]);
        else		
        exit(1);
   
 inicjalizacja tablicy reprezentującej siatkę stanów
   
    int siatka[256] =
    {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0,
        0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};
        
   inicjalizacja zmiennych typu int i zmiennych dla mpi i tablicy intów
   
    int sp, si, id, j, iter = 0;
 
    MPI_Init(&argc, &argv);
    MPI_Status status;
    MPI_Comm_size(MPI_COMM_WORLD, &sp);
    MPI_Comm_rank(MPI_COMM_WORLD, &id);
    
    int *tablica = (int*)malloc(ROZMIAR*((ROZMIAR/sp)+2)*sizeof(int));
    
główny for programu

    for (iteracje = 0; iteracje < si; iteracje++) {
    
 for przypisujący wartości z tablicy siatka do tablicy o nazwie tablica
    
     for (int i = id * (SIATKA / sp); i < (SIATKA / sp) * (id + 1); i++) {
                tablica[j] = siatka[i];
                j++;
            }
            
   if sprawdzający czy sp nie jest równe 1 jeśi tak to tworzy tablice intów 2 do wysyłanych danych i 2 do przyjmowanych danych. Następnie w zależności od reszty z dzielenia przez 2 zmiennej id, są przyisywane odpowiednie wartości do tablic i wysyłane lub odbierane. I dokonywane są obliczenia potrzebne do działania programu.
 
     if (sp != 1) {
                int tabPrzychodzaca1[ROZMIAR], tabPrzychodzaca2[ROZMIAR], tabWysylka1[ROZMIAR], tabWysylka2[ROZMIAR];
                if (id%2 == 0) {
                    for (int i=0; i<ROZMIAR; i++) 
                        tabWysylka1[i] = tablica[i + ROZMIAR];
                
                MPI_Send(&tabWysylka1, ROZMIAR, MPI_INT, MOD(id - 1, sp), 1, MPI_COMM_WORLD);

                for (int i = 0; i < ROZMIAR; i++) 
                    tabWysylka2[i] = tablica[(ROZMIAR * (ROZMIAR / sp)) + i];
                
                MPI_Send(&tabWysylka2, ROZMIAR, MPI_INT, MOD(id + 1, sp), 1, MPI_COMM_WORLD);
            }
            else {
                MPI_Recv(&tabPrzychodzaca2, ROZMIAR, MPI_INT, MOD(id + 1, sp), 1, MPI_COMM_WORLD, &status);
                MPI_Recv(&tabPrzychodzaca1, ROZMIAR, MPI_INT, MOD(id - 1, sp), 1, MPI_COMM_WORLD, &status);
            }
            if (id%2 == 0) {
                MPI_Recv(&tabPrzychodzaca2, ROZMIAR, MPI_INT, MOD(id + 1, sp), 1, MPI_COMM_WORLD, &status);
                MPI_Recv(&tabPrzychodzaca1, ROZMIAR, MPI_INT, MOD(id - 1, sp), 1, MPI_COMM_WORLD, &status);
            }
            else {
                for (int i=0; i<ROZMIAR; i++) 
                    tabWysylka1[i] = tablica[i+ROZMIAR];
                
                MPI_Send(&tabWysylka1, ROZMIAR, MPI_INT, MOD(id - 1, sp), 1, MPI_COMM_WORLD);

                for (int i = 0; i<ROZMIAR; i++) 
                    tabWysylka2[i] = tablica[(ROZMIAR * (ROZMIAR / sp)) + i];
                
                MPI_Send(&tabWysylka2, ROZMIAR, MPI_INT, MOD(id + 1, sp), 1, MPI_COMM_WORLD);
            }
            for (int i=0; i<ROZMIAR; i++) {
                tablica[i] = tabPrzychodzaca1[i];
                tablica[(ROZMIAR * ((ROZMIAR / sp) + 1)) + i] = tabPrzychodzaca2[i];
            }
        }
        else {
            for (int i = SIATKA; i < SIATKA + ROZMIAR; i++) 
                tablica[i - SIATKA] = siatka[i - ROZMIAR];
			
            for (int i=0; i<ROZMIAR; i++) 
                tablica[i + SIATKA + ROZMIAR] = siatka[i];
        }
        
  Inicjalizacja zmiennych typu int przypisywanie im wartości obliczanie wartości dla zmiennej obliczenia wykorzystując elementy tablicy o nazwie tablica. jest to obliczanie stanów danej komurki. Następnie przypisywana jest odpowiednia wartość do danej komurki. Na koniec zapisywane dane są do tablicy siatka.
        
        int *wynik = (int*)malloc(((ROZMIAR / sp) * ROZMIAR) * sizeof(int));

        for (int k = ROZMIAR; k < ROZMIAR*((ROZMIAR / sp)+1); k++) {
            int wszystkie_wiersze = ROZMIAR * (ROZMIAR / sp) + 2;
            int aktualny_wiersz = k / ROZMIAR;
            int kolumna = k % ROZMIAR;
            
            int pop_wiersz = MOD(aktualny_wiersz - 1, wszystkie_wiersze);
            int nast_wiersz = MOD(aktualny_wiersz + 1, wszystkie_wiersze);

            int pop_kolumna = MOD(kolumna - 1, ROZMIAR);
            int nast_kolumna = MOD(kolumna + 1, ROZMIAR);
            

            int obliczenia = tablica[pop_wiersz * ROZMIAR + pop_kolumna] + 
            tablica[pop_wiersz * ROZMIAR + kolumna] + 
            tablica[pop_wiersz * ROZMIAR + nast_kolumna] + 
            tablica[aktualny_wiersz * ROZMIAR + popa_kolumna] + 
            tablica[aktualny_wiersz * ROZMIAR + nast_kolumna] + 
            tablica[nast_wiersz * ROZMIAR + pop_kolumna] + 
            tablica[nast_wiersz * ROZMIAR + kolumna] + 
            tablica[nast_wiersz * ROZMIAR + nast_kolumna];
			
            if (tablica[k] == 1) {
                if (obliczenia < 2 && obliczenia > 3)
                    wynik[k - ROZMIAR] = 0;
                else
                    wynik[k - ROZMIAR] = 1;
            }
            else {
                if (obliczenia != 3)
                    wynik[k - ROZMIAR] = 0;
                else
                    wynik[k - ROZMIAR] = 1;
            }
        }

        j = 0;
        for (int i = id * (SIATKA / sp); i < (id + 1) * (SIATKA / sp); i++) {
            siatka[i] = wynik[j];
            j++;
        }
        
  Wyświtlnie siatki
        
        if (id == 0) 
            rysuj(siatka);
        
    }
    
  Zwolneienie pamięci z tablicy i wywołanie funkcji kończącej działanie pacy w trybie mpi.
    
     free(tablica);
    MPI_Finalize();
