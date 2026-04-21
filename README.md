#include <iostream>
#include <fstream>
#include <iomanip>
#include <sstream>
#include <map>
#include <vector>
using namespace std;

class Theatre
{
private:
    string movies[3] = {
        "Avengers Endgame",
        "Inception",
        "Interstellar"
    };

    map<string, vector<vector<char>>> dateSeats;
    string currentDate;

    int ticketPrice = 200;
    int ticketID;

public:

    Theatre()
    {
        ticketID = loadLastTicketID();
    }

    void initializeSeats(string date)
    {
        if(dateSeats.find(date) == dateSeats.end())
        {
            vector<vector<char>> seats(5, vector<char>(5, 'O'));
            dateSeats[date] = seats;
        }
    }

    int loadLastTicketID()
    {
        ifstream file("tickets.txt");
        string line;
        int lastID = 1000;

        while(getline(file,line))
        {
            size_t pos = line.find("TicketID:");
            if(pos != string::npos)
            {
                int id;
                sscanf(line.c_str(), "TicketID: %d", &id);
                lastID = id;
            }
        }

        file.close();
        return lastID;
    }

    void showMovies()
    {
        cout<<"\n======= NOW SHOWING =======\n";
        for(int i=0;i<3;i++)
            cout<<i+1<<". "<<movies[i]<<endl;
    }

    void showSeats()
    {
        cout<<"\nDate: "<<currentDate<<endl;
        cout<<"\nSeat Layout (O = Free, X = Booked)\n\n";
        cout<<"   1 2 3 4 5\n";

        for(int i=0;i<5;i++)
        {
            cout<<char('A'+i)<<"  ";
            for(int j=0;j<5;j++)
                cout<<dateSeats[currentDate][i][j]<<" ";
            cout<<endl;
        }
    }

    bool isValidSeat(char row, int col)
    {
        return (row >= 'A' && row <= 'E' && col >= 1 && col <= 5);
    }

    void bookTicket()
    {
        int movieChoice;
        string name;
        char row;
        int column;

        cout<<"Enter date (DD-MM-YYYY): ";
        cin>>currentDate;

        initializeSeats(currentDate);

        showMovies();

        cout<<"\nSelect movie (1-3): ";
        cin>>movieChoice;

        if(movieChoice < 1 || movieChoice > 3)
        {
            cout<<"Invalid movie selection!\n";
            return;
        }

        cin.ignore();

        cout<<"Enter full name: ";
        getline(cin, name);

        showSeats();

        cout<<"\nChoose seat\n";
        cout<<"Row (A-E): ";
        cin>>row;
        row = toupper(row);

        cout<<"Column (1-5): ";
        cin>>column;

        if(!isValidSeat(row, column))
        {
            cout<<"Invalid seat selection!\n";
            return;
        }

        int r = row - 'A';
        int c = column - 1;

        if(dateSeats[currentDate][r][c] == 'X')
        {
            cout<<"Seat already booked for this date!\n";
            return;
        }

        dateSeats[currentDate][r][c] = 'X';
        ticketID++;

        saveTicket(name, movieChoice, row, column, currentDate);
        printTicket(name, movieChoice, row, column);
    }

    void saveTicket(string name, int movieChoice, char row, int column, string date)
    {
        ofstream file("tickets.txt", ios::app);

        file<<"TicketID: "<<ticketID
            <<" | Name: "<<name
            <<" | Movie: "<<movies[movieChoice-1]
            <<" | Date: "<<date
            <<" | Seat: "<<row<<column
            <<" | Price: "<<ticketPrice<<endl;

        file.close();
    }

    void printTicket(string name, int movieChoice, char row, int column)
    {
        cout<<"\n========= TICKET =========\n";
        cout<<"Ticket ID : "<<ticketID<<endl;
        cout<<"Customer  : "<<name<<endl;
        cout<<"Movie     : "<<movies[movieChoice-1]<<endl;
        cout<<"Date      : "<<currentDate<<endl;
        cout<<"Seat      : "<<row<<column<<endl;
        cout<<"Price     : Rs "<<ticketPrice<<endl;
    }

    //  MODIFIED: Date-wise booking view
    void viewBookings()
    {
        string searchDate;
        cout<<"Enter date to view bookings (DD-MM-YYYY): ";
        cin>>searchDate;

        ifstream file("tickets.txt");
        string line;

        cout<<"\n===== BOOKINGS FOR "<<searchDate<<" =====\n";

        if(!file)
        {
            cout<<"No bookings found.\n";
            return;
        }

        bool found = false;

        while(getline(file,line))
        {
            if(line.find("Date: " + searchDate) != string::npos)
            {
                cout<<line<<endl;
                found = true;
            }
        }

        if(!found)
            cout<<"No bookings found for this date.\n";

        file.close();
    }

    //  MODIFIED: Date-wise revenue
    void revenueReport()
    {
        string searchDate;
        cout<<"Enter date for revenue report (DD-MM-YYYY): ";
        cin>>searchDate;

        ifstream file("tickets.txt");
        string line;
        int count = 0;

        while(getline(file,line))
        {
            if(line.find("Date: " + searchDate) != string::npos)
            {
                count++;
            }
        }

        file.close();

        cout<<"\n Total Tickets Sold on "<<searchDate<<": "<<count<<endl;
        cout<<" Total Revenue: Rs "<<count * ticketPrice<<endl;
    }

    void clearData()
    {
        ofstream file("tickets.txt", ios::trunc);
        file.close();

        ticketID = 1000;
        dateSeats.clear();

        cout<<"\nAll booking data cleared successfully!\n";
    }

    void adminPanel()
    {
        string password;

        cout<<"\nEnter Admin Password: ";
        cin>>password;

        if(password != "admin123")
        {
            cout<<"Access Denied!\n";
            return;
        }

        int choice;

        do
        {
            cout<<"\n======= ADMIN PANEL =======\n";
            cout<<"1. View Bookings (Date-wise)\n";
            cout<<"2. Revenue Report (Date-wise)\n";
            cout<<"3. Clear All Data\n";
            cout<<"4. Back\n";

            cout<<"Enter choice: ";
            cin>>choice;

            switch(choice)
            {
                case 1: viewBookings(); break;
                case 2: revenueReport(); break;
                case 3: clearData(); break;
                case 4: break;
                default: cout<<"Invalid option!\n";
            }

        } while(choice != 4);
    }
};

int main()
{
    Theatre system;
    int choice;

    do
    {
        cout<<"\n=================================\n";
        cout<<"   MOVIE TICKET BOOKING SYSTEM\n";
        cout<<"=================================\n";

        cout<<"1. Show Movies\n";
        cout<<"2. Book Ticket\n";
        cout<<"3. Admin Panel\n";
        cout<<"4. Exit\n";

        cout<<"Enter choice: ";
        cin>>choice;

        switch(choice)
        {
            case 1: system.showMovies(); break;
            case 2: system.bookTicket(); break;
            case 3: system.adminPanel(); break;
            case 4: cout<<"Thank you for using system!\n"; break;
            default: cout<<"Invalid option!\n";
        }

    } while(choice != 4);

    return 0;
}
