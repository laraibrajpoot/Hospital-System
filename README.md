#include <iostream>
#include <string>
#include <queue>
#include <stack>
#include <fstream>
#include <iomanip>

using namespace std;

// Patient Structure (Linked List)
struct Patient {
    int id;
    string name;
    int age;
    string disease;
    Patient* next;
};

Patient* head = NULL;

// Stack for patient history
stack<string> history;

// Priority Queue for emergency patients
priority_queue<pair<int, string>> emergencyQueue;

// Queue for doctor appointment
queue<string> doctorQueue;

// ---------- FILE HANDLING ----------
void savePatientToFile(Patient* p) {
    ofstream file("patients.txt", ios::app);
    file << p->id << "|" << p->name << "|" << p->age << "|" << p->disease << endl;
    file.close();
}

void saveHistoryToFile(const string& record) {
    ofstream file("history.txt", ios::app);
    file << record << endl;
    file.close();
}

void loadPatientsFromFile() {
    ifstream file("patients.txt");
    if (!file) return;

    Patient* temp;
    while (!file.eof()) {
        Patient* p = new Patient();
        char sep;

        file >> p->id;
        file >> sep;
        getline(file, p->name, '|');
        file >> p->age;
        file >> sep;
        getline(file, p->disease);

        if (file.fail()) break;

        p->next = NULL;

        if (!head) head = p;
        else {
            temp = head;
            while (temp->next) temp = temp->next;
            temp->next = p;
        }

        doctorQueue.push(p->name);
    }
    file.close();
}

// ---------- CORE FUNCTIONS ----------
void addPatient() {
    Patient* p = new Patient();

    cout << "\nEnter Patient ID: ";
    cin >> p->id;
    cin.ignore();

    cout << "Enter Name: ";
    getline(cin, p->name);

    cout << "Enter Age: ";
    cin >> p->age;
    cin.ignore();

    cout << "Enter Disease: ";
    getline(cin, p->disease);

    p->next = NULL;

    if (!head) head = p;
    else {
        Patient* temp = head;
        while (temp->next) temp = temp->next;
        temp->next = p;
    }

    doctorQueue.push(p->name);

    string record = "Added Patient: " + p->name;
    history.push(record);
    savePatientToFile(p);
    saveHistoryToFile(record);

    cout << "\n✔ Patient added successfully!\n";
}

void displayPatients() {
    if (!head) {
        cout << "\n❌ No patients available.\n";
        return;
    }

    cout << "\n================ PATIENT LIST ================\n";
    cout << left << setw(6) << "ID"
        << setw(20) << "Name"
        << setw(6) << "Age"
        << setw(20) << "Disease" << endl;
    cout << "---------------------------------------------\n";

    Patient* temp = head;
    while (temp) {
        cout << setw(6) << temp->id
            << setw(20) << temp->name
            << setw(6) << temp->age
            << setw(20) << temp->disease << endl;
        temp = temp->next;
    }
}

void addEmergency() {
    string name;
    int priority;

    cin.ignore();
    cout << "\nEnter Patient Name: ";
    getline(cin, name);

    cout << "Priority (1-High, 2-Medium, 3-Low): ";
    cin >> priority;

    emergencyQueue.push({ -priority, name });

    string record = "Emergency Added: " + name;
    history.push(record);
    saveHistoryToFile(record);

    cout << "🚨 Emergency patient added!\n";
}

void serveEmergency() {
    if (emergencyQueue.empty()) {
        cout << "\nNo emergency patients.\n";
        return;
    }

    cout << "\n🚑 Serving Emergency Patient: "
        << emergencyQueue.top().second << endl;

    emergencyQueue.pop();
}

void serveDoctor() {
    if (doctorQueue.empty()) {
        cout << "\nNo patients waiting for doctor.\n";
        return;
    }

    string name = doctorQueue.front();
    doctorQueue.pop();

    cout << "\n👨‍⚕️ Doctor is serving: " << name << endl;

    string record = "Doctor Served: " + name;
    history.push(record);
    saveHistoryToFile(record);
}

void showHistory() {
    if (history.empty()) {
        cout << "\nNo history available.\n";
        return;
    }

    cout << "\n============== HISTORY LOG ==============\n";
    stack<string> temp = history;
    while (!temp.empty()) {
        cout << temp.top() << endl;
        temp.pop();
    }
}

// ---------- MAIN ----------
int main() {
    loadPatientsFromFile();

    int choice;
    do {
        cout << "\n========== HOSPITAL MANAGEMENT ==========\n";
        cout << "1. Add Patient\n";
        cout << "2. Display Patients\n";
        cout << "3. Add Emergency Patient\n";
        cout << "4. Serve Emergency Patient\n";
        cout << "5. Serve Doctor Queue\n";
        cout << "6. Show History\n";
        cout << "0. Exit\n";
        cout << "Enter choice: ";
        cin >> choice;

        switch (choice) {
        case 1: addPatient(); break;
        case 2: displayPatients(); break;
        case 3: addEmergency(); break;
        case 4: serveEmergency(); break;
        case 5: serveDoctor(); break;
        case 6: showHistory(); break;
        case 0: cout << "\nSystem Closed Successfully.\n"; break;
        default: cout << "\n❌ Invalid choice!\n";
        }
    } while (choice != 0);

    return 0;
}
