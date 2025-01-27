#include <iostream>
#include <fstream>
#include <vector>
#include <queue>
#include <unordered_map>

using namespace std;


void wczytajGraf(const string& plikWejsciowy, unordered_map<int, vector<int>>& graf, unordered_map<int, int>& stopnieWejscia) {
    ifstream wejscie(plikWejsciowy);
    if (!wejscie.is_open()) {
        cerr << "Nie można otworzyć pliku wejściowego." << endl;
        exit(1);
    }

    int a, b;
    while (wejscie >> a >> b) {
        graf[a].push_back(b);
        stopnieWejscia[b]++;
        if (stopnieWejscia.find(a) == stopnieWejscia.end()) {
            stopnieWejscia[a] = 0;
        }
    }

    wejscie.close();
}


void zapiszWynik(const string& plikWyjsciowy, const vector<int>& wynik, bool czyMozliwe) {
    ofstream wyjscie(plikWyjsciowy);
    if (!wyjscie.is_open()) {
        cerr << "Nie można otworzyć pliku wyjściowego." << endl;
        exit(1);
    }

    if (czyMozliwe) {
        wyjscie << "<";
        for (size_t i = 0; i < wynik.size(); ++i) {
            wyjscie << wynik[i];
            if (i < wynik.size() - 1) wyjscie << ", ";
        }
        wyjscie << ">\n";
    }
    else {
        wyjscie << "Nie można ustalić kolejności wykonywania czynności." << endl;
    }

    wyjscie.close();
}


bool sortowanieTopologiczne(const unordered_map<int, vector<int>>& graf, unordered_map<int, int>& stopnieWejscia, vector<int>& wynik) {
    queue<int> kolejka;

    
    for (const auto& para : stopnieWejscia) {
        if (para.second == 0) {
            kolejka.push(para.first);
        }
    }

    while (!kolejka.empty()) {
        int wierzcholek = kolejka.front();
        kolejka.pop();
        wynik.push_back(wierzcholek);

        
        for (int sasiad : graf.at(wierzcholek)) {
            stopnieWejscia[sasiad]--;
            if (stopnieWejscia[sasiad] == 0) {
                kolejka.push(sasiad);
            }
        }
    }

    
    return wynik.size() == stopnieWejscia.size();
}

int main(int argc, char* argv[]) {
    if (argc != 5 || string(argv[1]) != "-i" || string(argv[3]) != "-u") {
        cout << "Użycie: " << argv[0] << " -i <plik_wejsciowy> -u <plik_wyjsciowy>" << endl;
        return 1;
    }

    string plikWejsciowy = argv[2];
    string plikWyjsciowy = argv[4];

    unordered_map<int, vector<int>> graf;
    unordered_map<int, int> stopnieWejscia;

    wczytajGraf(plikWejsciowy, graf, stopnieWejscia);

    vector<int> wynik;
    bool czyMozliwe = sortowanieTopologiczne(graf, stopnieWejscia, wynik);

    zapiszWynik(plikWyjsciowy, wynik, czyMozliwe);

    return 0;
}

