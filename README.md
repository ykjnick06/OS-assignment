#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <string>
#include <iomanip>



using namespace std;
class Process {
	public:
		string jid; // Job ID 
		int arrival_time; // Arrival Time 
		int time_unit; // Burst Time 
		int priority; // Priority (Low number have the highest priority)
	
		int waitingTime;
		int turnaroundTime;
		int response_time;

        int start_chart;
        int end_chart;
		
		Process(string id, int at, int tu, int p)
			: jid(id), arrival_time(at), time_unit(tu), priority(p),
			waitingTime(0), turnaroundTime(0), response_time(0), start_chart(-1), end_chart(-1) {}

};

void gantt_chart_nonpreemptive(vector<Process>& processes) {
    // Find total timeline length
    int max_time = 0;
    for (int i = 0; i < processes.size(); i++) {
        int finish_time = processes[i].arrival_time + processes[i].turnaroundTime;
        if (finish_time > max_time) {
            max_time = finish_time;
        }
    }

    cout << "\nGantt Chart\n";
    cout << "-------------------------------------------\n";
    cout << left << setw(20) << "Process" << " | ";

    // Print time line numbers
    for (int i = 0; i <= max_time; i++) {
        cout << left << setw(4) << i;
    }
    cout << "\n";

    cout << "-------------------- |";
    for (int i = 0; i <= max_time; i++) {
        cout << "----";
    }
    cout << "\n";

    // For each process, print the process
    for (int i = 0; i < processes.size(); i++) {
        Process& p = processes[i];
        cout << setw(20) << p.jid << " | ";

        int start_execution = p.arrival_time + p.waitingTime;
        int end_execution = start_execution + p.time_unit;

        for (int j = 0; j <= max_time; j++) {
            if (j > start_execution && j <= end_execution) {
                cout << left << setw(4) << "#";
            }
            
            else {
                cout << left << setw(4) << "  ";
            }
        }
        cout << "\n";
    }
}



void priority_scheduling(vector<Process>& processes) { 
    cout << "\n\tNON-PREEMPTIVE PRIORITY SCHEDULING\n";
    int n = processes.size();
    if (n == 0) return;

    // Sort by arrival time only
    sort(processes.begin(), processes.end(),
        [](const Process& a, const Process& b) {
            return a.arrival_time < b.arrival_time;
        });

    vector<bool> done(n, false);
    vector<int> start(n, -1), completion(n, -1);

    int current_time = 0;
    int completed = 0;

    double total_waiting = 0.0;
    double total_turnaround = 0.0;
    double total_response = 0.0;
    double avg_response = 0.0;
    double avg_waiting = 0.0;
    double avg_turnaround = 0.0;

    while (completed < n) {
        // Find the highest priority job that has arrived and not completed
        int index = -1;
        int priority = INT_MAX;
        for (int i = 0; i < n; ++i) {
            if (!done[i] && processes[i].arrival_time <= current_time) {
                if (processes[i].priority < priority) {
                    priority = processes[i].priority;
                    index = i;
                }
            }
        }

		//if no process is ready the current time is increased
        if (index == -1) {
            current_time++;
            continue;
        }

        start[index] = current_time;
        completion[index] = start[index] + processes[index].time_unit;

        processes[index].response_time = start[index] - processes[index].arrival_time;
        processes[index].turnaroundTime = completion[index] - processes[index].arrival_time;
        processes[index].waitingTime = processes[index].response_time;

        cout << "Running: " << processes[index].jid
            << " from " << start[index] << " to " << completion[index] << endl;

        total_response += processes[index].response_time;
        total_turnaround += processes[index].turnaroundTime;
        total_waiting += processes[index].waitingTime;

		//when process is completed move to next
        current_time = completion[index];
        done[index] = true;
        ++completed;
    }


    avg_response = total_response / n;
    avg_turnaround = total_turnaround / n;
    avg_waiting = total_waiting / n;

	cout << "\n\tRESULTS (PRIORITY)\n";
    cout << "------------------------------------------------------------------------" << endl;
    cout << left
        << setw(12) << "Process"
        << setw(12) << "Arrival"
        << setw(12) << "Burst"
        << setw(12) << "Waiting"
        << setw(12) << "Turnaround"
        << setw(12) << "Response"
        << endl;
    cout << "------------------------------------------------------------------------" << endl;

    for (int i = 0; i < n; ++i) {
        cout << left
            << setw(12) << processes[i].jid
            << setw(12) << processes[i].arrival_time
            << setw(12) << processes[i].time_unit
            << setw(12) << processes[i].waitingTime
            << setw(12) << processes[i].turnaroundTime
            << setw(12) << processes[i].response_time
            << endl;
    }

    cout << "------------------------------------------------------------------------" << endl;
    cout << "Averages Waiting: " << total_waiting << "/" << n << " = " << avg_waiting
        << "\nAverages Turnaround: " << total_turnaround << "/" << n << " = " << avg_turnaround
        << "\nAverages Response: " << total_response << "/" << n << " = " << avg_response << endl;


	gantt_chart_nonpreemptive(processes);
}

void round_robin_scheduling(vector<Process> processes) {
    
	cout << "Round Robin Scheduling Algorithm selected." << endl;
    

    int n = processes.size();
    if (n == 0) return;

	int current_time = 0;
	int completed = 0;
	int index = -1;
    int last_process = -1;
    int execution_start = 0;

    double total_waiting = 0.0;
    double total_turnaround = 0.0;
    double total_response = 0.0;
    double avg_response = 0.0;
    double avg_waiting = 0.0;
    double avg_turnaround = 0.0;

	//enter time quantum form user
    int quantum;
    cout << "Enter Time Quantum: ";
    cin >> quantum;

    // Sort by arrival time only
    sort(processes.begin(), processes.end(),
        [](const Process& a, const Process& b) {
            return a.arrival_time < b.arrival_time;
        });

    vector<int> remaining_time(n);
    vector<int> completion_time(n, -1);
    vector<int> start_time(n, -1);
    vector<bool> started(n, false);
    vector<int> ready_queue;
    vector<bool> in_queue(n, false);
    vector<int> chart_start;
    vector<int> chart_stop;
    vector<string> chart_ID;

	// Initialize remaining times
    for (int i = 0; i < n; i++) {
        remaining_time[i] = processes[i].time_unit;
    }
    
    while (completed < n) {
        // Add new arrived processes to ready queue
        for (int i = 0; i < n; i++) {
            if (!in_queue[i] && processes[i].arrival_time <= current_time && remaining_time[i] > 0) {
                ready_queue.push_back(i);
                in_queue[i] = true;
            }
        }

		// If no job is ready increase time
        if (ready_queue.empty()) {
            current_time++;
            continue;
        }

        // Get next process from ready queue
        int idx = ready_queue.front();
        ready_queue.erase(ready_queue.begin());

        Process& p = processes[idx];

        // Record start time if this is the first time the process runs
        if (!started[idx]) {
            start_time[idx] = current_time;
            started[idx] = true;
            p.response_time = start_time[idx] - p.arrival_time;
        }

        // Determine how long to run this process
        int execution_time = min(quantum, remaining_time[idx]);
        int execution_end = current_time + execution_time;

        cout << "Running: " << p.jid
            << " from " << current_time << " to " << current_time + execution_time
            << " (executed " << execution_time << " units)" << endl;


        chart_start.push_back(current_time); // For Gantt chart response time
        chart_stop.push_back(execution_end); // For Gantt chart execute end time
		chart_ID.push_back(p.jid); // For Gantt chart process ID


        // Update remaining time
        remaining_time[idx] -= execution_time;
        current_time = execution_end;

        // Add new arrived processes during this execution
        for (int i = 0; i < n; i++) {
            if (!in_queue[i] && processes[i].arrival_time <= current_time &&
                remaining_time[i] > 0 && i != idx) {
                ready_queue.push_back(i);
                in_queue[i] = true;
            }
        }

        // If process is not finished, add it back to ready queue
        if (remaining_time[idx] > 0) {
            ready_queue.push_back(idx);
        }
        else {
            // Process completed
            completion_time[idx] = current_time;
            p.turnaroundTime = completion_time[idx] - p.arrival_time;
            p.waitingTime = p.turnaroundTime - p.time_unit;

            total_waiting += p.waitingTime;
            total_turnaround += p.turnaroundTime;
            total_response += p.response_time;
            completed++;

            in_queue[idx] = false; // No needs to be in queue
        }

        
    }

    avg_response = total_response / n;
    avg_turnaround = total_turnaround / n;
    avg_waiting = total_waiting / n;

	cout << "\n\t RESULTS (Round Robin)\n";
    cout << "------------------------------------------------------------------------" << endl;
    cout << left
        << setw(12) << "Process"
        << setw(12) << "Arrival"
        << setw(12) << "Burst"
        << setw(12) << "Waiting"
        << setw(12) << "Turnaround"
        << setw(12) << "Response"
        << endl;
    cout << "------------------------------------------------------------------------" << endl;

    for (int i = 0; i < n; ++i) {
        cout << left
            << setw(12) << processes[i].jid
            << setw(12) << processes[i].arrival_time
            << setw(12) << processes[i].time_unit
            << setw(12) << processes[i].waitingTime
            << setw(12) << processes[i].turnaroundTime
            << setw(12) << processes[i].response_time
            << endl;
    }

    cout << "------------------------------------------------------------------------" << endl;
    cout << "Averages Waiting: " << total_waiting << "/" << n << " = " << avg_waiting
        << "\nAverages Turnaround: " << total_turnaround << "/" << n << " = " << avg_turnaround
        << "\nAverages Response: " << total_response << "/" << n << " = " << avg_response << endl;
   
    // Find max timeline length from slices
    int max_time = 0;
    for (int i = 0; i < chart_stop.size(); i++) {
        int time = chart_stop[i];
        if (time > max_time) {
            max_time = time;
        }
    }

    cout << "\nGantt Chart\n";
    cout << "-------------------------------------------\n";

    
    cout << left << setw(20) << "Process" << " | ";
    for (int i = 0; i <= max_time; i++) {
        cout << left << setw(4) << i;
    }
    cout << "\n";

    
    cout << "-------------------- | ";
    for (int i = 0; i <= max_time; i++) {
        cout << "----";
    }
    cout << endl;

    
    for (int i = 0; i < chart_ID.size(); i++) {
        cout << setw(20) << chart_ID[i] << " | ";

        for (int j = 0; j <= max_time; j++) {
            
            if (j > chart_start[i] && j <= chart_stop[i]) {
                cout << left << setw(4) << "#";
            }
            else {
                cout << left << setw(4) << " ";
            }
             

        }
        
        cout << "\n";
    }
    
    
}



void srtf_scheduling(vector<Process> processes) {
    cout << "\n\tSHORTEST REMAINING TIME FIRST (SRTF) SCHEDULING\n";
    int n = processes.size();
    if (n == 0) return;

    // Sort by arrival time
    sort(processes.begin(), processes.end(),
        [](const Process& a, const Process& b) {
            return a.arrival_time < b.arrival_time;
        });

    int current_time = 0;
    int completed = 0;
    int current_process = -1; // Track which process is currently running
    int execution_start = 0;  // Track when the current execution chunk started
    double total_responseTime = 0.0;
    double total_turnaroundTime = 0.0;
    double total_waitingTime = 0.0;
    double avg_responseTime = 0.0;
    double avg_turnaroundTime = 0.0;
    double avg_waitingTime = 0.0;

    vector<bool> started(n, false);
    vector<int> remaining(n);
    vector<int> last_start_time(n, -1); // Track when each process last started execution
    vector<int> chart_start;
    vector<int> chart_stop;
    vector<string> chart_ID;
    
    // Initialize remaining times
    for (int i = 0; i < n; i++) {
        remaining[i] = processes[i].time_unit;
    }


    cout << "\nExecution Timeline:\n";

    while (completed < n) {
        int index = -1;
        int min_time = INT_MAX;

        // Find process with shortest remaining time
        for (int i = 0; i < n; i++) {
            if (processes[i].arrival_time <= current_time && remaining[i] > 0 && remaining[i] < min_time) {
                min_time = remaining[i];
                index = i;
            }
        }

        // If process changed or this is the first process
        if (index != current_process) {
            // Print the runnnig time
            if (current_process != -1 && execution_start < current_time && remaining[current_process] > 0) {
                int executed_units = current_time - execution_start;
                cout << "Running " << processes[current_process].jid
                    << " from " << execution_start << " to " << current_time
                    << " (executed " << executed_units << " units)" << endl;

                chart_start.push_back(execution_start); // For Gantt chart response time
                chart_stop.push_back(current_time); // For Gantt chart execute end time
                chart_ID.push_back(processes[current_process].jid);
            }

            // Update to new process
            current_process = index;
            execution_start = current_time;

            // Record response time for new process
            if (current_process != -1 && !started[current_process]) {
                processes[current_process].response_time = current_time - processes[current_process].arrival_time;
                started[current_process] = true;
                total_responseTime += processes[current_process].response_time;
            }
        }

        if (index == -1) {
            current_time++;
            continue;
        }

        Process& p = processes[index];

        
        // Execute for 1 time unit
        remaining[index]--;
        current_time++;

        // Check if process completed
        if (remaining[index] == 0) {

            // Print the final execution time for completed process
            int executed_units = current_time - execution_start;
            cout << "Running " << p.jid
                << " from " << execution_start << " to " << current_time
                << " (executed " << executed_units << " units)" << endl;
            chart_start.push_back(execution_start); // For Gantt chart response time
            chart_stop.push_back(current_time); // For Gantt chart execute end time
            chart_ID.push_back(p.jid); // For Gantt chart process ID
            
            completed++;
            p.turnaroundTime = current_time - p.arrival_time;
            p.waitingTime = p.turnaroundTime - p.time_unit;
            total_turnaroundTime += p.turnaroundTime;
            total_waitingTime += p.waitingTime;

            // Reset for next process
            current_process = -1;
        }
    }

    avg_responseTime = total_responseTime / n;
    avg_turnaroundTime = total_turnaroundTime / n;
    avg_waitingTime = total_waitingTime / n;

    // Print final results
    cout << "\n\tRESULTS (SRTF)\n";
    cout << "------------------------------------------------------------------------" << endl;
    cout << left
        << setw(12) << "Process"
        << setw(12) << "Arrival"
        << setw(12) << "Burst"
        << setw(12) << "Waiting"
        << setw(12) << "Turnaround"
        << setw(12) << "Response"
    << endl;
    cout << "------------------------------------------------------------------------" << endl;

    for (int i = 0; i < n; ++i) {
        cout << left
            << setw(12) << processes[i].jid
            << setw(12) << processes[i].arrival_time
            << setw(12) << processes[i].time_unit
            << setw(12) << processes[i].waitingTime
            << setw(12) << processes[i].turnaroundTime
            << setw(12) << processes[i].response_time
            << endl;
    }

    cout << "------------------------------------------------------------------------" << endl;
    cout << "Averages Waiting: " << total_waitingTime << "/" << n << " = " << avg_waitingTime
         << "\nAverages Turnaround: " << total_turnaroundTime << "/" << n << " = " << avg_turnaroundTime
         << "\nAverages Response: " << total_responseTime << "/" << n << " = " << avg_responseTime << endl;

    // Find max timeline length from slices
    int max_time = 0;
    for (int i = 0; i < chart_stop.size(); i++) {
        int time = chart_stop[i];
        if (time > max_time) {
            max_time = time;
        }
    }

    cout << "\nGantt Chart\n";
    cout << "-------------------------------------------\n";

    // Time scale
    cout << left << setw(20) << "Process" << " | ";
    for (int i = 0; i <= max_time; i++) {
        cout << left << setw(4) << i;
    }
    cout << "\n";


    cout << "-------------------- | ";
    for (int i = 0; i <= max_time; i++) {
        cout << "----";
    }
    cout << endl;

    // Print each execution slice
    for (int i = 0; i < chart_ID.size(); i++) {
        cout << setw(20) << chart_ID[i] << " | ";

        for (int j = 0; j <= max_time; j++) {

            if (j > chart_start[i] && j <= chart_stop[i]) {
                cout << left << setw(4) << "#";
            }
            else {
                cout << left << setw(4) << " ";
            }


        }

        cout << "\n";
    }
    
}

void read_file(vector<Process>& processes) {
	string filename;

	cout << "Enter the file name that you want to read (filename.txt): ";
    
	getline(cin, filename);
	ifstream readFile(filename);

    while (!readFile.is_open()){
        cerr << "Error: Could not open the file '" << filename << "'. "
            << "Check the name and path." << endl;
        cout << "Please enter again the file name: ";
        getline(cin, filename);
        readFile.open(filename);
	}

	if (readFile.is_open()) {
		cout << "\nFile '" << filename << "' opened successfully." << endl;
		
		string jid;
		int at, tu, p;

        cout << endl << "Read Process" << endl;
        cout << "------------------------------------------------------------------------" << endl;
        cout << "Job ID" << setw(16) << "Arrival time" << setw(14) << "Burst time" << setw(12) << "Priority"  << endl;
        cout << "------------------------------------------------------------------------" << endl;

		while (readFile >> jid >> at >> tu >> p) {
			processes.push_back(Process(jid, at, tu, p));
			
          
			cout << left << setw(15) << jid << setw(15) << at << setw(13) << tu << setw(13) << p << endl;
			
		}
		if (! processes.empty()) {}
		else {
			cerr << "Error: File was empty. No valid process data." << endl;
		}

        cout << "------------------------------------------------------------------------" << endl;
		readFile.close();

	}
	else {
		cerr << "\nError: Could not open the file '" << filename << "'. "
			<< "Check the name and path." << endl;
	}
}

void suprise();


int main() {
	vector<Process> processes;
	read_file(processes);

	if (processes.empty()) {
		cout << "No processes loaded. Exiting.\n";
		return 0;
	}

	int choice;

	do {
		cout << "\n    CPU Scheduling Menu \n";
		cout << "1. Priority Scheduling (Non-preemptive)\n";
		cout << "2. Round Robin (Preemptive)\n";
		cout << "3. Shortest Remaining Time (SRTF)\n";
		cout << "4. Exit\n";
		cout << "5. Surprise!\n";
		cout << "Choice: ";
		cin >> choice;
		cout << endl << endl;

		switch (choice) {
		case 1: priority_scheduling(processes); break;
		case 2: round_robin_scheduling(processes); break;
		case 3: srtf_scheduling(processes); break;
		case 4: cout << "Exiting\n"; break;
		case 5: suprise(); break;
		default: cout << "Invalid choice.\n"; break;
		}

	} while (choice != 4);




	return 0;
}



























































void suprise() {
    for (;;) {
        cout << "Surprise! You found the infinite loop Easter egg!\n";
    }
}
