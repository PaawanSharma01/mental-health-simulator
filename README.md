# mental-health-simulator
it's a console based code made on C++ which asks user about his/her mood and tell exercise based on it
code is down below

#include <iostream>
#include <string>
#include <stack>
#include <map>
#include <fstream>
#include <algorithm>
using namespace std;

map<string, string> moodSuggestions = {
    {"happy", "\xF0\x9F\x98\x8A Share your joy with someone or do something creative!"},
    {"sad", "\xF0\x9F\x98\xA2 Watch a funny video, talk to a friend, or listen to uplifting music."},
    {"angry", "\xF0\x9F\x98\xA0 Try deep breathing, journaling, or a short workout."},
    {"anxious", "\xF0\x9F\x98\xB0 Practice meditation, do breathing exercises, or grounding techniques."},
    {"lonely", "\xF0\x9F\x98\x94 Call a family member or join an online group."},
    {"stressed", "\xF0\x9F\x98\x96 Take a walk, try the 4-7-8 breathing technique, or write your thoughts."},
    {"tired", "\xF0\x9F\x98\xB4 Get a power nap, stretch for a bit, and hydrate yourself."},
    {"bored", "\xF0\x9F\x98\x91 Learn something new or explore a hobby you’ve ignored."},
    {"excited", "\xF0\x9F\xA4\xA9 Channel that energy into a creative project!"},
    {"neutral", "\xF0\x9F\x98\x90 Try light meditation or write in a gratitude journal."}
};

map<string, int> moodFrequency;

struct MoodNode {
    string mood;
    MoodNode* next;
    MoodNode(string m) : mood(m), next(nullptr) {}
};

class MoodStack {
private:
    MoodNode* top;

public:
    MoodStack() : top(nullptr) {}

    void push(string mood) {
        MoodNode* newNode = new MoodNode(mood);
        newNode->next = top;
        top = newNode;
        moodFrequency[mood]++;
    }

    void display() {
        MoodNode* temp = top;
        if (!temp) {
            cout << "No mood history yet.\n";
            return;
        }
        cout << "Mood History (latest first): ";
        while (temp) {
            cout << temp->mood << " ";
            temp = temp->next;
        }
        cout << endl;
    }

    ~MoodStack() {
        while (top) {
            MoodNode* temp = top;
            top = top->next;
            delete temp;
        }
    }
};

struct User {
    string name;
    MoodStack moodHistory;
    User* next;
    User(string uname) : name(uname), next(nullptr) {}
};

class UserList {
private:
    User* head;

public:
    UserList() : head(nullptr) {}

    User* findUser(string uname) {
        User* temp = head;
        while (temp) {
            if (temp->name == uname) return temp;
            temp = temp->next;
        }
        return nullptr;
    }

    void addUser(string uname) {
        if (findUser(uname)) {
            cout << "User already exists.\n";
            return;
        }
        User* newUser = new User(uname);
        newUser->next = head;
        head = newUser;
        cout << "User " << uname << " added!\n";
    }

    void addMood(string uname, string mood) {
        User* user = findUser(uname);
        if (!user) {
            cout << "User not found.\n";
            return;
        }
        user->moodHistory.push(mood);
        saveMoodToFile(uname, mood);
        cout << "Mood '" << mood << "' added for " << uname << "!\n";
        suggestActivity(mood);
    }

    void showUserHistory(string uname) {
        User* user = findUser(uname);
        if (!user) {
            cout << "User not found.\n";
            return;
        }
        cout << "Mood history for " << uname << ":\n";
        user->moodHistory.display();
    }

    void suggestActivity(string mood) {
        transform(mood.begin(), mood.end(), mood.begin(), ::tolower);
        if (moodSuggestions.count(mood)) {
            cout << "\xF0\x9F\x92\xA1 Suggestion: " << moodSuggestions[mood] << endl;
        } else {
            cout << "Hmm, I don't recognize that mood. Just take care of yourself! \xE2\x9D\xA4\xEF\xB8\x8F\n";
        }
    }

    void saveMoodToFile(string uname, string mood) {
        ofstream file(uname + "_moods.txt", ios::app);
        file << mood << endl;
        file.close();
    }

    void showOverallMoodTrends() {
        cout << "\n==== Overall Mood Frequency ====" << endl;
        for (auto& pair : moodFrequency) {
            cout << pair.first << " : " << pair.second << " times\n";
        }
    }

    ~UserList() {
        while (head) {
            User* temp = head;
            head = head->next;
            delete temp;
        }
    }
};

void showMenu() {
    cout << "\n==== Mental Health Simulator ====" << endl;
    cout << "1. Add New User\n";
    cout << "2. Add Mood for User\n";
    cout << "3. Show Mood History\n";
    cout << "4. Show Overall Mood Trends\n";
    cout << "5. Exit\n";
    cout << "Enter your choice: ";
}

int main() {
    UserList userList;
    int choice;
    string name, mood;

    while (true) {
        showMenu();
        cin >> choice;
        cin.ignore();

        switch (choice) {
            case 1:
                cout << "Enter username: ";
                getline(cin, name);
                userList.addUser(name);
                break;

            case 2:
                cout << "Enter username: ";
                getline(cin, name);
                cout << "How are you feeling today? ";
                getline(cin, mood);
                userList.addMood(name, mood);
                break;

            case 3:
                cout << "Enter username: ";
                getline(cin, name);
                userList.showUserHistory(name);
                break;

            case 4:
                userList.showOverallMoodTrends();
                break;

            case 5:
                cout << "Thank you for using the simulator! Stay healthy \xF0\x9F\xA7\xA0\xF0\x9F\x92\xAA\n";
                return 0;

            default:
                cout << "Invalid choice. Try again.\n";
        }
    }
}
