# Computer-Science-311---Assignment-2-AI-Survey-Program
/*
 * Name: Kareem Abu Humra
 * Class: Computer Science 311 AI Survey Program
 * Assignment: Political Party Survey Program
 * Date: [Insert the date here]
 */

#include <iostream>
#include <vector>
#include <string>
#include <map>
#include <fstream>
#include <sstream>
#include <limits>  // Include this header for std::numeric_limits

// Struct to hold each survey question and its options
struct Question {
    std::string questionText;  // Text of the question
    std::vector<std::string> options;  // Possible answer choices
};

// Struct to store the user's response to a question
struct Response {
    int questionIndex;  // Index of the question
    int optionIndex;  // Index of the chosen option
};

// Enum to represent different political parties
enum PoliticalParty { DEMOCRAT, REPUBLICAN, LIBERTARIAN, INDEPENDENT };

// Map to store the names of the political parties
std::map<PoliticalParty, std::string> partyNames = {
    {DEMOCRAT, "Democrat"},
    {REPUBLICAN, "Republican"},
    {LIBERTARIAN, "Libertarian"},
    {INDEPENDENT, "Independent"}
};

// Class to handle the survey logic
class Survey {
private:
    std::vector<Question> questions;  // List of survey questions
    std::vector<Response> responses;  // User responses to the questions
    std::map<PoliticalParty, int> scores;  // Scores for each political party
    std::map<PoliticalParty, std::vector<std::vector<int>>> historicalData;  // Historical response data

public:
    // Constructor to initialize the survey
    Survey() {
        // Initialize scores for each party to zero
        scores[DEMOCRAT] = 0;
        scores[REPUBLICAN] = 0;
        scores[LIBERTARIAN] = 0;
        scores[INDEPENDENT] = 0;

        // Add survey questions
        questions.push_back({"What should the government do to help the poor?",
                             {"1. Make it easier to apply for assistance",
                              "2. Allow parents to use education funds for charter schools",
                              "3. Create welfare to work programs",
                              "4. Nothing"}});
        questions.push_back({"What is your stance on healthcare?",
                             {"1. Support universal healthcare",
                              "2. Support private healthcare options",
                              "3. Support a mix of both",
                              "4. No government involvement in healthcare"}});
        questions.push_back({"What are your views on gun control?",
                             {"1. Support strict gun control laws",
                              "2. Support the right to bear arms",
                              "3. Support background checks",
                              "4. No restrictions on gun ownership"}});
        questions.push_back({"How do you view climate change?",
                             {"1. Support strong environmental regulations",
                              "2. Support balanced regulations",
                              "3. Climate change is not a priority",
                              "4. Do not believe in climate change"}});
        questions.push_back({"What is your opinion on immigration policies?",
                             {"1. Support more open borders",
                              "2. Support stricter immigration controls",
                              "3. Support balanced immigration policies",
                              "4. No strong opinion"}});
        questions.push_back({"How should taxes be managed?",
                             {"1. Increase taxes on the wealthy",
                              "2. Reduce taxes for all",
                              "3. Keep taxes as they are",
                              "4. Abolish income tax"}});
        questions.push_back({"What is your stance on education?",
                             {"1. Increase funding for public schools",
                              "2. Support school voucher programs",
                              "3. Support homeschooling",
                              "4. Reduce government involvement in education"}});
        questions.push_back({"What are your views on military spending?",
                             {"1. Increase military spending",
                              "2. Maintain current levels",
                              "3. Reduce military spending",
                              "4. Abolish military"}});
        questions.push_back({"How should the government handle drug policy?",
                             {"1. Legalize and regulate all drugs",
                              "2. Decriminalize drug use",
                              "3. Maintain current drug policies",
                              "4. Increase penalties for drug use"}});
        questions.push_back({"What are your views on same-sex marriage?",
                             {"1. Fully support",
                              "2. Oppose",
                              "3. Support civil unions only",
                              "4. No opinion"}});
        // More questions can be added as needed...

        // Load historical data
        loadHistoricalData();
    }

    // Method to start the survey and collect responses
    void startSurvey() {
        // Loop through each question
        for (int i = 0; i < questions.size(); ++i) {
            std::cout << questions[i].questionText << std::endl;
            // Display the options for the current question
            for (int j = 0; j < questions[i].options.size(); ++j) {
                std::cout << questions[i].options[j] << std::endl;
            }

            int userResponse;
            while (true) {
                std::cin >> userResponse;  // Get the user's response
                if (userResponse > 0 && userResponse <= questions[i].options.size() && !std::cin.fail()) {
                    break;
                } else {
                    std::cin.clear();  // Clear the error flag
                    std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');  // Discard invalid input
                    std::cout << "Invalid option. Please enter a number between 1 and " << questions[i].options.size() << ": ";
                }
            }

            responses.push_back({i, userResponse - 1});  // Store the response
        }

        guessParty();  // Guess the user's political party based on the scores
    }

    // Method to load historical data from files
    void loadHistoricalData() {
        for (const auto& party : partyNames) {
            std::string fileName = party.second + ".txt";
            std::ifstream file(fileName);
            if (file.is_open()) {
                std::string line;
                while (getline(file, line)) {
                    std::istringstream iss(line);
                    int questionIndex, optionIndex;
                    char comma;
                    if (iss >> questionIndex >> comma >> optionIndex) {
                        historicalData[party.first][questionIndex].push_back(optionIndex);
                    }
                }
                file.close();
            }
        }
    }

    // Method to calculate and apply weights based on historical data
    void calculateAndApplyWeights() {
        std::map<PoliticalParty, std::vector<double>> weights;

        for (const auto& partyData : historicalData) {
            PoliticalParty party = partyData.first;
            const std::vector<std::vector<int>>& data = partyData.second;

            weights[party].resize(questions.size());

            for (size_t i = 0; i < data.size(); ++i) {
                const std::vector<int>& responses = data[i];
                std::map<int, int> responseCount;
                for (int response : responses) {
                    responseCount[response]++;
                }

                double totalResponses = responses.size();
                for (const auto& count : responseCount) {
                    weights[party][i] += (count.second / totalResponses);
                }
            }
        }

        // Apply weights to current survey responses
        for (const auto& response : responses) {
            int questionIndex = response.questionIndex;
            int optionIndex = response.optionIndex;

            for (const auto& weight : weights) {
                scores[weight.first] += weight.second[questionIndex];
            }
        }
    }

    // Method to guess the user's political party based on scores
    void guessParty() {
        calculateAndApplyWeights();

        PoliticalParty guessedParty = DEMOCRAT;
        int maxScore = 0;
        // Find the party with the highest score
        for (const auto& score : scores) {
            if (score.second > maxScore) {
                maxScore = score.second;
                guessedParty = score.first;
            }
        }
        // Display the guessed party
        std::cout << "Based on your responses, we guess your political party is: " << partyNames[guessedParty] << std::endl;

        // Ask the user to confirm their actual party affiliation
        std::cout << "Which political party do you affiliate with?\n1. Democrat\n2. Republican\n3. Libertarian\n4. Independent\n";
        int actualParty;
        while (true) {
            std::cin >> actualParty;
            if (actualParty > 0 && actualParty <= 4 && !std::cin.fail()) {
                break;
            } else {
                std::cin.clear();  // Clear the error flag
                std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');  // Discard invalid input
                std::cout << "Invalid option. Please enter a number between 1 and 4: ";
            }
        }

        std::cout << "You selected: " << partyNames[static_cast<PoliticalParty>(actualParty - 1)] << std::endl;
        // Save the response to a file
        saveResponse(static_cast<PoliticalParty>(actualParty - 1));
    }

    // Method to save the response to a file
    void saveResponse(PoliticalParty party) {
        std::string fileName = partyNames[party] + ".txt";
        std::ofstream file;
        file.open(fileName, std::ios_base::app);  // Open the file in append mode
        for (const auto& response : responses) {
            file << response.questionIndex << "," << response.optionIndex << "\n";  // Save only indices
        }
        file.close();
    }
};

// Main function to run the survey
int main() {
    Survey survey;  // Create a Survey object
    survey.startSurvey();  // Start the survey
    return 0;
}
