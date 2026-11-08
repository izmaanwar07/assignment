
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <memory>
#include <iomanip>
#include <limits>
#include <stdexcept>
using namespace std;

// Abstract class
class Advisory
{
public:
    virtual void giveAdvice() const = 0;
    virtual ~Advisory() {}
};

// Base class
class Crop
{
protected:
    string name, season;
    double area, yieldPerAcre;
    static int cropCount;

public:
    // Default constructor
    Crop()
    {
        name = "Unknown";
        season = "Unknown";
        area = 1;
        yieldPerAcre = 0;
        cropCount++;
    }
    Crop(string n, double a, string s, double y)
    {
        if (a <= 0 || y < 0)
            throw invalid_argument("Invalid area or yield!");
            name = n;
        area = a;
        season = s;
        yieldPerAcre = y;
        cropCount++;
    }
    virtual ~Crop()
    {
        cropCount--;
    }
    virtual double totalYield() const
    {
        return area * yieldPerAcre;
    }
 virtual void display() const
    {
        cout << fixed << setprecision(2);
        cout << "\nCrop Name: " << name
             << "\nArea: " << area << " acres"
             << "\nSeason: " << season
             << "\nYield per Acre: " << yieldPerAcre
             << "\nTotal Yield: " << totalYield() << endl;
    }
    bool operator>(const Crop& other) const
    {
        return totalYield() > other.totalYield();
    }
    virtual void save(ofstream& file) const = 0;

   static int getCropCount()
    {
        return cropCount;
    }
};

int Crop::cropCount = 0;

class CerealCrop : public Crop, public Advisory
{
    string fertilizerType;

public:
    CerealCrop(string n, double a, string s, double y,
               string f)
        : Crop(n, a, s, y)
    {
        fertilizerType = f;
    }

   void display() const override
    {
        cout << "\n--- Cereal Crop ---";
        Crop::display();
        cout << "Fertilizer: " << fertilizerType << endl;
    }

   void giveAdvice() const override
    {
        cout << "\nAdvice for " << name
             << ": Use balanced fertilizer, remove weeds, "
             << "and monitor crop growth regularly.\n";
    }

   void save(ofstream& file) const override
    {
        file << "Cereal|" << name << "|" << area << "|"
             << season << "|" << yieldPerAcre << "|"
             << fertilizerType << "|\n";
    }
};

class VegetableCrop : public Crop, public Advisory
{
    int irrigationFrequency;

public:
    VegetableCrop(string n, double a, string s, double y,
                  int irrigation)
        : Crop(n, a, s, y)
    {
        if (irrigation <= 0)
            throw invalid_argument(
                "Irrigation frequency must be positive!");

 irrigationFrequency = irrigation;
    }

  void display() const override
    {
        cout << "\n--- Vegetable Crop ---";
        Crop::display();
        cout << "Irrigation: " << irrigationFrequency
             << " times per week\n";
    }

  void giveAdvice() const override
    {
        cout << "\nAdvice for " << name
             << ": Maintain suitable soil moisture, "
             << "avoid overwatering, and check for pests.\n";
    }

  void save(ofstream& file) const override
    {
        file << "Vegetable|" << name << "|" << area << "|"
             << season << "|" << yieldPerAcre << "|"
             << irrigationFrequency << "|\n";
    }
};

double getDouble(string message, bool allowZero)
{
    double value;

  while (true)
    {
        cout << message;

  if (cin >> value &&
            value >= 0 &&
            (allowZero || value > 0))
        {
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            return value;
        }

  cout << "Invalid input. Please try again.\n";
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
}

int getInt(string message)
{
    int value;
while (true)
    {
        cout << message;

  if (cin >> value && value > 0)
        {
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            return value;
        }

   cout << "Enter a positive integer.\n";
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
}

string getText(string message)
{
    string value;

 while (true)
    {
        cout << message;
        getline(cin, value);

  if (!value.empty() && value.find('|') == string::npos)
            return value;

  cout << "Enter valid text without the | character.\n";
    }
}

int main()
{
    vector<unique_ptr<Crop>> crops;
    int choice;
   do
    {
        cout << "\n===== SMART FARM MANAGEMENT SYSTEM =====\n";
        cout << "1. Add Cereal Crop\n";
        cout << "2. Add Vegetable Crop\n";
        cout << "3. Display All Crops\n";
        cout << "4. Compare Two Crops\n";
        cout << "5. Get Farming Advice\n";
        cout << "6. Save Crop Data\n";
        cout << "7. Retrieve Saved Data\n";
        cout << "8. Display Crop Count\n";
        cout << "9. Exit\n";

   cout << "Enter choice: ";

  if (!(cin >> choice))
        {
            cout << "Invalid menu input.\n";
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            continue;
        }

   cin.ignore(numeric_limits<streamsize>::max(), '\n');

   try
        {
            if (choice == 1 || choice == 2)
            {
                string n = getText("Enter crop name: ");
                double a = getDouble("Enter area in acres: ", false);
                string s = getText("Enter season: ");
                double y = getDouble("Enter yield per acre: ", true);

  if (choice == 1)
                {
                    string f = getText("Enter fertilizer type: ");

   crops.push_back(
                        make_unique<CerealCrop>(n, a, s, y, f));
                }
                else
                {
                    int irrigation = getInt(
                        "Enter irrigation times per week: ");

   crops.push_back(
                        make_unique<VegetableCrop>(
                            n, a, s, y, irrigation));
                }
cout << "Crop added successfully.\n";
            }
            else if (choice == 3)
            {
                if (crops.empty())
                {
                    cout << "No crops recorded.\n";
                }
                else
                {
                    for (const auto& crop : crops)
                        crop->display();
                }
            }
            else if (choice == 4)
            {
                if (crops.size() < 2)
                {
                    cout << "Add at least two crops first.\n";
                    continue;
                }
  int i, j;

cout << "Enter first crop number (1 to "
                     << crops.size() << "): ";
                if (!(cin >> i))
                {
                    cin.clear();
                    cin.ignore(numeric_limits<streamsize>::max(), '\n');
                    cout << "Invalid crop number.\n";
                    continue;
                }
  cout << "Enter second crop number (1 to "
                     << crops.size() << "): ";
                if (!(cin >> j))
                {
                    cin.clear();
                    cin.ignore(numeric_limits<streamsize>::max(), '\n');
                    cout << "Invalid crop number.\n";
                    continue;
                }
                cin.ignore(numeric_limits<streamsize>::max(), '\n');

   if (i < 1 || i > (int)crops.size() ||
                    j < 1 || j > (int)crops.size() || i == j)
                {
                    cout << "Invalid crop selection.\n";
                    continue;
                }

  if (*crops[i - 1] > *crops[j - 1])
                    cout << "Crop " << i << " has the higher yield.\n";
                else if (*crops[j - 1] > *crops[i - 1])
                    cout << "Crop " << j << " has the higher yield.\n";
                else
                    cout << "Both crops have equal yields.\n";
            }
            else if (choice == 5)
            {
                if (crops.empty())
                {
                    cout << "Add a crop first.\n";
                    continue;
                }

  int i;
                cout << "Enter crop number (1 to "
                     << crops.size() << "): ";
   if (!(cin >> i))
                {
                    cin.clear();
                    cin.ignore(numeric_limits<streamsize>::max(), '\n');
                    cout << "Invalid crop number.\n";
                    continue;
                }

  cin.ignore(numeric_limits<streamsize>::max(), '\n');

 if (i < 1 || i > (int)crops.size())
                {
                    cout << "Invalid crop selection.\n";
                    continue;
                }

  Advisory* advice =
                    dynamic_cast<Advisory*>(crops[i - 1].get());

  if (advice != nullptr)
                    advice->giveAdvice();
            }
            else if (choice == 6)
            {
                ofstream file("crops.txt", ios::app);

  if (!file)
                    throw runtime_error("Cannot open file for saving.");
   for (const auto& crop : crops)
                    crop->save(file);

  file.close();
                cout << "Crop data saved to crops.txt.\n";
            }
            else if (choice == 7)
            {
                ifstream file("crops.txt");

  if (!file)
                {
                    cout << "No saved file found.\n";
                    continue;
                }

 string line;
                cout << "\n--- Saved Crop Records ---\n";

while (getline(file, line))
                    cout << line << '\n';

   file.close();
            }
            else if (choice == 8)
            {
                cout << "Current crop objects: "
                     << Crop::getCropCount() << endl;
            }
            else if (choice == 9)
            {
                cout << "Exiting Smart Farm Management System.\n";
            }
            else
            {
                cout << "Invalid choice. Select 1 to 9.\n";
            }
        }
        catch (const exception& e)
        {
            cout << "Error: " << e.what() << endl;
        }
  } while (choice != 9);

  return 0;
}
