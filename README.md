#include <iostream>
#include <iomanip>
#include <vector>
#include <queue>
#include <climits>
#include <string>

using namespace std;

// Structure Definitions
struct Process {
    int pid;
    int arrivalTime;
    int burstTime;
    int remainingTime;
    int completionTime;
    int waitingTime;
    int turnaroundTime;
    bool completed;
    int startTime;
    int priority;
};

struct GanttChartEntry {
    int pid;
    int startTime;
    int endTime;
};

// Function Declarations
void displayMenu();
string getAlgorithmName(int choice);
bool confirmChoice(string message);
void processesInput(Process processes[], int n, bool isPriorityScheduling = false);
void FCFS(Process processes[], int n, vector<GanttChartEntry> &ganttChart);
void SJF_NonPreemptive(Process processes[], int n, vector<GanttChartEntry> &ganttChart);
void SJF_Preemptive(Process processes[], int n, vector<GanttChartEntry> &ganttChart);
void RoundRobin(Process processes[], int n, int timeQuantum, vector<GanttChartEntry> &ganttChart);
void Priority_NonPreemptive(Process processes[], int n, vector<GanttChartEntry> &ganttChart);
void Priority_Preemptive(Process processes[], int n, vector<GanttChartEntry> &ganttChart);
void ganttChartPrint(vector<GanttChartEntry> ganttChart);
void printProcessTable(Process processes[], int n);
void performanceMetrics(Process processes[], int n);

// Main Function
int main() {
    int n, choice, timeQuantum;

    do {
        displayMenu();

        cout << "\nEnter your choice: ";
        while (!(cin >> choice) || choice < 1 || choice > 7) {
            cout << "Invalid choice! Please enter a number between 1 and 7: ";
            cin.clear();
            cin.ignore(1000, '\n');
        }

        // Exit program
        if (choice == 7) {
            cout << "\nExiting the program...\n";
            break;
        }

        // Confirm the algorithm choice
        if (!confirmChoice("\nYou have chosen: " + getAlgorithmName(choice) + ". Do you want to continue?"))
            continue;

        // Enter the number of processes with confirmation
        do {
            cout << "\nEnter the number of processes: ";
            while (!(cin >> n) || n <= 0) {
                cout << "Invalid input! Please enter a positive number: ";
                cin.clear();
                cin.ignore(1000, '\n');
            }
        } while (!confirmChoice("You have entered " + to_string(n) + " processes. Do you want to continue?"));

        Process processes[n];
        vector<GanttChartEntry> ganttChart;

        bool requiresPriority = (choice == 4 || choice == 5);
        processesInput(processes, n, requiresPriority);

        if (choice == 6) {
            cout << "Enter Time Quantum: ";
            while (!(cin >> timeQuantum) || timeQuantum <= 0) {
                cout << "Invalid input! Please enter a positive number: ";
                cin.clear();
                cin.ignore(1000, '\n');
            }
        }

        // Call selected scheduling algorithm
        switch (choice) {
            case 1: FCFS(processes, n, ganttChart); break;
            case 2: SJF_NonPreemptive(processes, n, ganttChart); break;
            case 3: SJF_Preemptive(processes, n, ganttChart); break;
            case 4: Priority_NonPreemptive(processes, n, ganttChart); break;
            case 5: Priority_Preemptive(processes, n, ganttChart); break;
            case 6: RoundRobin(processes, n, timeQuantum, ganttChart); break;
        }

        // Display results
        ganttChartPrint(ganttChart);
        printProcessTable(processes, n);
        performanceMetrics(processes, n);

    } while (true);

    return 0;
}

// Function Definitions

// Display Menu
void displayMenu() {
    cout << "\n========== Intelligent CPU Scheduler Simulator ==========\n";
    cout << "\n1. First Come First Serve (FCFS)\n";
    cout << "2. Shortest Job First (SJF Non-Preemptive)\n";
    cout << "3. Shortest Job First (SJF Preemptive)\n";
    cout << "4. Priority Scheduling (Non-Preemptive)\n";
    cout << "5. Priority Scheduling (Preemptive)\n";
    cout << "6. Round Robin\n";
    cout << "7. Exit\n";
    cout << "\n";
}

// Get Algorithm Name
string getAlgorithmName(int choice) {
    switch (choice) {
        case 1: return "First Come First Serve (FCFS)";
        case 2: return "Shortest Job First (Non-Preemptive)";
        case 3: return "Shortest Job First (Preemptive)";
        case 4: return "Priority Scheduling (Non-Preemptive)";
        case 5: return "Priority Scheduling (Preemptive)";
        case 6: return "Round Robin";
        default: return "";
    }
}

// Confirm Choice
bool confirmChoice(string message) {
    char confirm;
    while (true) {
        cout << message << " (Y/N): ";
        cin >> confirm;
        if (confirm == 'y' || confirm == 'Y') return true;
        if (confirm == 'n' || confirm == 'N') return false;
        cout << "Invalid input! Please enter 'Y' for Yes or 'N' for No: ";
    }
}

// Input Handling
void processesInput(Process processes[], int n, bool isPriorityScheduling) {
    cout << "\nEnter process details:\n";
    for (int i = 0; i < n; i++) {
        processes[i].pid = i + 1;
        cout << "P" << i + 1 << ":\n";

        // Arrival Time Validation
        cout << " Arrival Time: ";
        while (!(cin >> processes[i].arrivalTime) || processes[i].arrivalTime < 0) {
            cout << " Invalid input! Please enter a valid non-negative number for Arrival Time: ";
            cin.clear();
            cin.ignore(1000, '\n');
        }

        // Burst Time Validation
        cout << " Burst Time: ";
        while (!(cin >> processes[i].burstTime) || processes[i].burstTime <= 0) {
            cout << " Invalid input! Please enter a valid positive number for Burst Time: ";
            cin.clear();
            cin.ignore(1000, '\n');
        }

        // Priority Validation (only for priority scheduling)
        if (isPriorityScheduling) {
            cout << " Priority: ";
            while (!(cin >> processes[i].priority) || processes[i].priority < 0) {
                cout << " Invalid input! Please enter a valid non-negative number for Priority: ";
                cin.clear();
                cin.ignore(1000, '\n');
            }
        } else {
            processes[i].priority = 0; // Default priority
        }
    }
}

// FCFS Algorithm
void FCFS(Process processes[], int n, vector<GanttChartEntry> &ganttChart) {
    int currentTime = processes18n(processes[0].arrivalTime); // Track CPU time
    for (int i = 0; i < n; i++) {
        // Handle CPU idle time
        if (processes[i].arrivalTime > currentTime) {
            ganttChart.push_back({-1, currentTime, processes[i].arrivalTime}); // IDLE block
            currentTime = processes[i].arrivalTime; // Move time forward
        }

        
        processes[i].completionTime = currentTime + processes[i].burstTime;
        processes[i].turnaroundTime = processes[i].completionTime - processes[i].arrivalTime;
        processes[i].waitingTime = processes[i].turnaroundTime - processes[i].burstTime;

        // Add to Gantt Chart
        ganttChart.push_back({processes[i].pid, currentTime, processes[i].completionTime});

        // Update current time
        currentTime = processes[i].completionTime;
    }
}

// SJF Non-Preemptive Algorithm
void SJF_NonPreemptive(Process processes[], int n, vector<GanttChartEntry> &ganttChart) {
    int currentTime = processes[0].arrivalTime, completedProcesses = 0;
    for (int i = 0; i < n; i++) {
        processes[i].completed = false;
    }

    while (completedProcesses < n) {
        int minBurst = INT_MAX, selected = -1;
        // Find the shortest job that has arrived
        for (int i = 0; i < n; i++) {
            if (!processes[i].completed && processes[i].arrivalTime <= currentTime && processes[i].burstTime < minBurst) {
                minBurst = processes[i].burstTime;
                selected = i;
            }
        }

        if (selected == -1) {
            // If no process is ready, CPU is idle
            int nextArrival = INT_MAX;
            for (int i = 0; i < n; i++) {
                if (!processes[i].completed && processes[i].arrivalTime > currentTime) {
                    nextArrival = min(nextArrival, processes[i].arrivalTime);
                }
            }
            ganttChart.push_back({-1, currentTime, nextArrival});
            currentTime = nextArrival;
        } else {
            // Process execution
            ganttChart.push_back({processes[selected].pid, currentTime, currentTime + processes[selected].burstTime});
            processes[selected].completionTime = currentTime + processes[selected].burstTime;
            processes[selected].turnaroundTime = processes[selected].completionTime - processes[selected].arrivalTime;
            processes[selected].waitingTime = processes[selected].turnaroundTime - processes[selected].burstTime;
            processes[selected].completed = true;
            currentTime = processes[selected].completionTime;
            completedProcesses++;
        }
    }
}

// SJF Preemptive Algorithm
void SJF_Preemptive(Process processes[], int n, vector<GanttChartEntry> &ganttChart) {
    int currentTime = processes[0].arrivalTime, completedProcesses = 0;
    int lastExecutedPid = -1, timeSliceStart = 0;

    for (int i = 0; i < n; i++) {
        processes[i].remainingTime = processes[i].burstTime;
        processes[i].completed = false;
    }

    while (completedProcesses < n) {
        int shortestIndex = -1;
        int minRemainingTime = INT_MAX;

        // Find the process with the shortest remaining time that has arrived
        for (int i = 0; i < n; i++) {
            if (!processes[i].completed && processes[i].arrivalTime <= currentTime && processes[i].remainingTime < minRemainingTime) {
                minRemainingTime = processes[i].remainingTime;
                shortestIndex = i;
            }
        }

        // If no process is available, CPU stays idle
        if (shortestIndex == -1) {
            if (lastExecutedPid != -1) {
                ganttChart.push_back({lastExecutedPid, timeSliceStart, currentTime}); // Close last process
            }
            if (ganttChart.empty() || ganttChart.back().pid != -1) {
                ganttChart.push_back({-1, currentTime, currentTime + 1}); // Mark idle
            } else {
                ganttChart.back().endTime++; // Extend idle period
            }
            lastExecutedPid = -1;
            currentTime++;
            continue;
        }

        // Process switching: Record the last executed process in Gantt Chart
        if (processes[shortestIndex].pid != lastExecutedPid) {
            if (lastExecutedPid != -1) {
                ganttChart.push_back({lastExecutedPid, timeSliceStart, currentTime}); // Close previous process
            }
            timeSliceStart = currentTime;
            lastExecutedPid = processes[shortestIndex].pid;
        }

        // Execute the process for one unit time
        processes[shortestIndex].remainingTime--;
        currentTime++;

        // If the process completes
        if (processes[shortestIndex].remainingTime == 0) {
            ganttChart.push_back({processes[shortestIndex].pid, timeSliceStart, currentTime});
            processes[shortestIndex].completionTime = currentTime;
            processes[shortestIndex].turnaroundTime = processes[shortestIndex].completionTime -processes[shortestIndex].arrivalTime;
            processes[shortestIndex].waitingTime = processes[shortestIndex].turnaroundTime - processes[shortestIndex].burstTime;
            processes[shortestIndex].completed = true;
            completedProcesses++;
            lastExecutedPid = -1;
        }
    }
}

// Priority Non-Preemptive Algorithm
void Priority_NonPreemptive(Process processes[], int n, vector<GanttChartEntry> &ganttChart) {
    int currentTime = processes[0].arrivalTime;
    int completedProcesses = 0;

    // Initialize all processes
    for (int i = 0; i < n; i++) {
        processes[i].completed = false;
        processes[i].remainingTime = processes[i].burstTime;
    }

    while (completedProcesses < n) {
        int selectedIndex = -1;
        int highestPriority = INT_MAX;

        // Find the highest priority available process
        for (int i = 0; i < n; i++) {
            if (!processes[i].completed && processes[i].arrivalTime <= currentTime) {
                // Lower priority number = higher priority
                if (processes[i].priority < highestPriority) {
                    highestPriority = processes[i].priority;
                    selectedIndex = i;
                }
                // Tie-breaker: Earlier arrival time
                else if (processes[i].priority == highestPriority && processes[i].arrivalTime < processes[selectedIndex].arrivalTime) {
                    selectedIndex = i;
                }
            }
        }

        if (selectedIndex == -1) { // No process is available, CPU is idle
            if (!ganttChart.empty() && ganttChart.back().pid == -1) {
                ganttChart.back().endTime++; // Extend last idle block
            } else {
                ganttChart.push_back({-1, currentTime, currentTime + 1}); // Start new idle block
            }
            currentTime++; // Move time forward
        } else {
            // If there was an idle period before, ensure it ends properly
            if (!ganttChart.empty() && ganttChart.back().pid == -1) {
                ganttChart.back().endTime = currentTime; // Close idle block
            }

            // Execute the selected process to completion
            processes[selectedIndex].startTime = currentTime;
            ganttChart.push_back({processes[selectedIndex].pid, currentTime, currentTime + processes[selectedIndex].burstTime});
            currentTime += processes[selectedIndex].burstTime;

            // Calculate metrics
            processes[selectedIndex].completionTime = currentTime;
            processes[selectedIndex].turnaroundTime = processes[selectedIndex].completionTime - processes[selectedIndex].arrivalTime;
            processes[selectedIndex].waitingTime = processes[selectedIndex].turnaroundTime - processes[selectedIndex].burstTime;
            processes[selectedIndex].completed = true;
            completedProcesses++;
        }
    }
}

// Priority Preemptive Algorithm
void Priority_Preemptive(Process processes[], int n, vector<GanttChartEntry> &ganttChart) {
    int currentTime = processes[0].arrivalTime, completedProcesses = 0;
    int lastExecutedPid = -1, timeSliceStart = 0;

    // Initialize remaining time for all processes
    for (int i = 0; i < n; i++) {
        processes[i].remainingTime = processes[i].burstTime;
        processes[i].completed = false;
    }

    while (completedProcesses < n) {
        int highestPriorityIndex = -1;
        int highestPriority = INT_MAX;

        // Find the process with the highest priority that has arrived
        for (int i = 0; i < n; i++) {
            if (!processes[i].completed && processes[i].arrivalTime <= currentTime && processes[i].priority < highestPriority) {
                highestPriority = processes[i].priority;
                highestPriorityIndex = i;
            }
        }

        // If no process is available, CPU stays idle
        if (highestPriorityIndex == -1) {
            if (lastExecutedPid != -1) {
                ganttChart.push_back({lastExecutedPid, timeSliceStart, currentTime}); // Close last process
            }
            if (ganttChart.empty() || ganttChart.back().pid != -1) {
                ganttChart.push_back({-1, currentTime, currentTime + 1}); // Mark idle
            } else {
                ganttChart.back().endTime++; // Extend idle period
            }
            lastExecutedPid = -1;
            currentTime++;
            continue;
        }

        // Process switching: Record the last executed process in Gantt Chart
        if (processes[highestPriorityIndex].pid != lastExecutedPid) {
            if (lastExecutedPid != -1) {
                ganttChart.push_back({lastExecutedPid, timeSliceStart, currentTime}); // Close previous process
            }
            timeSliceStart = currentTime;
            lastExecutedPid = processes[highestPriorityIndex].pid;
        }

        // Execute the process for one unit time
        processes[highestPriorityIndex].remainingTime--;
        currentTime++;

        // If the process completes
        if (processes[highestPriorityIndex].remainingTime == 0) {
            ganttChart.push_back({processes[highestPriorityIndex].pid, timeSliceStart, currentTime});
            processes[highestPriorityIndex].completionTime = currentTime;
            processes[highestPriorityIndex].turnaroundTime = processes[highestPriorityIndex].completionTime - processes[highestPriorityIndex].arrivalTime;
            processes[highestPriorityIndex].waitingTime = processes[highestPriorityIndex].turnaroundTime - processes[highestPriorityIndex].burstTime;
            processes[highestPriorityIndex].completed = true;
            completedProcesses++;
            lastExecutedPid = -1;
        }
    }
}

// Round Robin Algorithm
void RoundRobin(Process processes[], int n, int quantum, vector<GanttChartEntry> &ganttChart) {
    if (quantum <= 0) {
        cerr << "Error: Time quantum must be positive\n";
        return;
    }

    queue<int> readyQueue;
    vector<int> remainingTime(n);
    vector<bool> isQueued(n, false);
    int currentTime = processes[0].arrivalTime, completedProcesses = 0;

    // Initialize
    for (int i = 0; i < n; i++) {
        remainingTime[i] = processes[i].burstTime;
        if (processes[i].arrivalTime == currentTime) {
            readyQueue.push(i);
            isQueued[i] = true;
        }
    }

    while (completedProcesses < n) {
        if (readyQueue.empty()) {
            // Handle idle CPU time
            int nextArrival = INT_MAX;
            for (int i = 0; i < n; i++) {
                if (remainingTime[i] > 0) {
                    nextArrival = min(nextArrival, processes[i].arrivalTime);
                }
            }
            if (nextArrival == INT_MAX) break; // All done

            // Add idle period to Gantt chart
            if (currentTime < nextArrival) {
                ganttChart.push_back({-1, currentTime, nextArrival});
            }
            currentTime = nextArrival;

            // Queue arriving processes
            for (int i = 0; i < n; i++) {
                if (!isQueued[i] && remainingTime[i] > 0 && processes[i].arrivalTime <= currentTime) {
                    readyQueue.push(i);
                    isQueued[i] = true;
                }
            }
            continue;
        }

        // Process execution
        int idx = readyQueue.front();
        readyQueue.pop();
        isQueued[idx] = false;

        int execTime = min(quantum, remainingTime[idx]);
        ganttChart.push_back({processes[idx].pid, currentTime, currentTime + execTime});
        currentTime += execTime;
        remainingTime[idx] -= execTime;

        // Check new arrivals during execution
        for (int i = 0; i < n; i++) {
            if (!isQueued[i] && remainingTime[i] > 0 && processes[i].arrivalTime > currentTime - execTime && processes[i].arrivalTime <= currentTime) {
                readyQueue.push(i);
                isQueued[i] = true;
            }
        }

        // Requeue if not finished
        if (remainingTime[idx] > 0) {
            readyQueue.push(idx);
            isQueued[idx] = true;
        } else {
            processes[idx].completionTime = currentTime;
            processes[idx].turnaroundTime = currentTime - processes[idx].arrivalTime;
            processes[idx].waitingTime = processes[idx].turnaroundTime - processes[idx].burstTime;
            completedProcesses++;
        }
    }
}

// Gantt Chart Visualization
void ganttChartPrint(vector<GanttChartEntry> ganttChart) {
    if (ganttChart.empty()) return;

    cout << "\nGantt Chart:\n";
    cout << " ";
    for (const auto& entry : ganttChart) {
        cout << "-------";
        (void)entry;
    }
    cout << "\n|";

    // Process IDs
    for (const auto& entry : ganttChart) {
        if (entry.pid == -1) {
            cout << " IDLE |"; // Mark idle periods
        } else {
            cout << " P" << entry.pid << " |";
        }
    }

    // Bottom border
    cout << "\n ";
    for (const auto& entry : ganttChart) {
        cout << "-------";
        (void)entry;
    }

    // Time labels (handles idle start time)
    cout << "\n" << ganttChart[0].startTime;
    for (const auto& entry : ganttChart) {
        cout << setw(7) << entry.endTime;
    }
    cout << "\n";
}

// Process Table Display
void printProcessTable(Process processes[], int n) {
    cout << "\nProcess Table:\n";
    cout << "-------------------------------------------------------------------------------------------------\n";
    cout << "| PID | Arrival Time | Burst Time | Priority | Completion Time | Turnaround Time | Waiting Time |\n";
    cout << "-------------------------------------------------------------------------------------------------\n";

    for (int i = 0; i < n; i++) {
        cout << "| " << setw(3) << processes[i].pid
             << " | " << setw(12) << processes[i].arrivalTime
             << " | " << setw(10) << processes[i].burstTime
             << " | " << setw(8) << processes[i].priority
             << " | " << setw(15) << processes[i].completionTime
             << " | " << setw(15) << processes[i].turnaroundTime
             << " | " << setw(12) << processes[i].waitingTime << " |\n";
    }
    cout << "-------------------------------------------------------------------------------------------------\n";
}

// Performance Metrics
void performanceMetrics(Process processes[], int n) {
    if (n == 0) {
        cout << "\nNo processes available.\n";
        return;
    }

    float totalTAT = 0, totalWT = 0;
    for (int i = 0; i < n; i++) {
        totalTAT += processes[i].turnaroundTime;
        totalWT += processes[i].waitingTime;
    }

    cout << "\nPerformance Metrics\n";
    cout << "---------------------------------------\n";
    cout << "|        Metric            |   Value |\n";
    cout << "---------------------------------------\n";
    cout << "| Average Turn Around Time | " << setw(5) << fixed << setprecision(2) << totalTAT / n << " ms |\n";
    cout << "| Average Waiting Time     | " << setw(5) << fixed << setprecision(2) << totalWT / n << " ms |\n";
    cout << "---------------------------------------\n";
}
