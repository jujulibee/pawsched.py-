# pawsched.py-

# PawSched: Console-Based Appointment Scheduling and Rescheduling System
# for a Veterinary Clinic (Group 5)

import datetime

appointments = []   # each item in this list is ONE appointment (a dictionary)
vets = ["Dr. A", "Dr. B"]
services = ["Check-up", "Vaccination", "Deworming", "Grooming"]
time_slots = ["8:00 AM", "9:00 AM", "10:00 AM", "11:00 AM", "1:00 PM",
              "2:00 PM", "3:00 PM", "4:00 PM", "5:00 PM"]
# ---------------------------------------------------------
# CONSOLE DESIGN
# ---------------------------------------------------------
RESET = "\033[0m"
BOLD = "\033[1m"
CYAN = "\033[96m"
GREEN = "\033[92m"
YELLOW = "\033[93m"
RED = "\033[91m"
MAGENTA = "\033[95m"

def clear_screen():
    print("\033[2J\033[H", end="")

def print_header(title, color=CYAN):
    print(color + BOLD)
    print("╔══════════════════════════════════════════════════════╗")
    print("║                    PAWSCHED                          ║")
    print("║        Veterinary Appointment System                 ║")
    print("╠══════════════════════════════════════════════════════╣")
    print("║  " + title.center(50) + "  ║")
    print("╚══════════════════════════════════════════════════════╝")
    print(RESET)

def print_menu_item(number, text):
    print(CYAN + "   [" + str(number) + "] " + RESET + text)
# ---------------------------------------------------------
# HELPER FUNCTIONS
# ---------------------------------------------------------
def show_menu():
    print()
    print("===== PAWSCHED: VET APPOINTMENT SYSTEM =====")
    print("  [1] Add Appointment")
    print("  [2] View All Appointments")
    print("  [3] Search Appointment")
    print("  [4] Reschedule Appointment")
    print("  [5] Cancel Appointment")
    print("  [6] Exit")

def choose_from_list(title, options):
    """Shows a numbered list and returns the option the user picked."""
    print()
    print(title)
    for i in range(len(options)):
        print("  " + str(i + 1) + ". " + options[i])

    while True:
        choice = input("Enter number: ")
        if choice.isdigit() and 1 <= int(choice) <= len(options):
            return options[int(choice) - 1]
        print("Invalid choice. Please try again.")

def ask_text(question):
    """Keeps asking until the user types something (not empty)."""
    while True:
        answer = input(question).strip()
        if answer != "":
            return answer
        print("This cannot be empty.")

def ask_date(question):
    """Keeps asking until the user enters a valid date that is today or later.
    Returns the date as text in YYYY-MM-DD format."""
    while True:
        text = input(question).strip()

        try:
            chosen = datetime.datetime.strptime(text, "%Y-%m-%d").date()
        except ValueError:
            print(RED + "✗ Invalid date. Use the format YYYY-MM-DD (example 2026-10-25)." + RESET)
            continue

        if chosen < datetime.date.today():
            print(RED + "✗ You cannot book a date in the past. Please enter today or a future date." + RESET)
            continue

        return chosen.strftime("%Y-%m-%d")

def is_taken(vet, date, time, skip_id=0):
    """Returns True if this vet already has an active appointment
    on the same date and time. This is what prevents double booking."""
    for appt in appointments:
        if (appt["vet"] == vet and appt["date"] == date
                and appt["time"] == time
                and appt["status"] != "Cancelled"
                and appt["id"] != skip_id):
            return True
    return False

def find_by_id(appt_id):
    """Looks for an appointment with this ID. Returns None if not found."""
    for appt in appointments:
        if appt["id"] == appt_id:
            return appt
    return None

def print_appointment(appt):
    print("ID: " + str(appt["id"]) + " | Owner: " + appt["owner"]
          + " | Contact: " + appt["contact"] + " | Pet: " + appt["pet"])
    print("     Service: " + appt["service"] + " | Vet: " + appt["vet"]
          + " | Date: " + appt["date"] + " | Time: " + appt["time"]
          + " | Status: " + appt["status"])

# ---------------------------------------------------------
# MAIN FEATURES
# ---------------------------------------------------------
def add_appointment():
    clear_screen()
    print_header("ADD APPOINTMENT", GREEN)
    owner = ask_text("Owner name: ")

    # Require exactly 11 digits for the contact number
    while True:
        contact = input("Contact number (11 digits): ").strip()

        if contact.isdigit() and len(contact) == 11:
            break

        print(RED + "✗ Invalid contact number. Please enter exactly 11 digits." + RESET)

    pet = ask_text("Pet name: ")
    service = choose_from_list("Choose a service:", services)
    vet = choose_from_list("Choose a veterinarian:", vets)
    date = ask_date("Date (YYYY-MM-DD, example 2026-10-25): ")
    time = choose_from_list("Choose a time slot:", time_slots)

    # Check for double booking before saving
    if is_taken(vet, date, time):
        print("\nSorry, " + vet + " is already booked on " + date + " at "
              + time + ". Please try another time.")
        return

    new_appt = {
        "id": len(appointments) + 1,
        "owner": owner,
        "contact": contact,
        "pet": pet,
        "service": service,
        "vet": vet,
        "date": date,
        "time": time,
        "status": "Scheduled"
    }
    appointments.append(new_appt)
    print("\n" + GREEN + BOLD + "✓ Appointment saved!" + RESET)
    print("  Your appointment ID is " + str(new_appt["id"]))


def view_appointments():
    clear_screen()
    print_header("ALL APPOINTMENTS", MAGENTA)
    if len(appointments) == 0:
        print("No appointments yet.")
        return
    for appt in appointments:
        print_appointment(appt)
        print()

def search_appointment():
    clear_screen()
    print_header("SEARCH APPOINTMENT", MAGENTA)
    keyword = ask_text("Enter owner name or pet name: ").lower()
    found = False
    for appt in appointments:
        if keyword in appt["owner"].lower() or keyword in appt["pet"].lower():
            print_appointment(appt)
            print()
            found = True
    if not found:
        print("No appointment found.")

def reschedule_appointment():
    clear_screen()
    print_header("RESCHEDULE APPOINTMENT", GREEN)
    id_text = ask_text("Enter appointment ID: ")
    if not id_text.isdigit():
        print("Invalid ID.")
        return

    appt = find_by_id(int(id_text))
    if appt is None:
        print("Appointment not found.")
        return
    if appt["status"] == "Cancelled":
        print("This appointment is already cancelled.")
        return

    print("Current appointment:")
    print_appointment(appt)

    new_date = ask_date("New date (YYYY-MM-DD): ")
    new_time = choose_from_list("Choose a new time slot:", time_slots)

    # skip_id = this appointment's own ID, so it does not conflict with itself
    if is_taken(appt["vet"], new_date, new_time, appt["id"]):
        print("\nSorry, that slot is already taken. Nothing was changed.")
        return

    appt["date"] = new_date
    appt["time"] = new_time
    appt["status"] = "Rescheduled"
    print("\n" + GREEN + BOLD + "✓ Appointment rescheduled successfully!" + RESET)

def cancel_appointment():
    clear_screen()
    print_header("CANCEL APPOINTMENT", GREEN)
    id_text = ask_text("Enter appointment ID: ")
    if not id_text.isdigit():
        print("Invalid ID.")
        return

    appt = find_by_id(int(id_text))
    if appt is None:
        print("Appointment not found.")
        return
    if appt["status"] == "Cancelled":
        print("This appointment is already cancelled.")
        return

    appt["status"] = "Cancelled"
    print("\n" + YELLOW + BOLD + "✓ Appointment cancelled." + RESET)
    print("  The time slot is now available again.")

# ---------------------------------------------------------
# OWNER / STAFF ACCOUNT
# ---------------------------------------------------------
OWNER_USERNAME = "Admin"
OWNER_PASSWORD = "pawschedg5"
# ---------------------------------------------------------
# ROLE MENUS AND LOGIN
# ---------------------------------------------------------
def show_role_menu(role):
    clear_screen()

    if role == "Client":
        print_header("CLIENT MENU", GREEN)
        print("  Manage your pet's veterinary appointment.\n")
        print_menu_item(1, "Add Appointment")
        print_menu_item(2, "Reschedule Appointment")
        print_menu_item(3, "Cancel Appointment")
        print_menu_item(4, "Exit")
    else:
        print_header("OWNER / STAFF MENU", MAGENTA)
        print("  Manage and search clinic appointments.\n")
        print_menu_item(1, "View All Appointments")
        print_menu_item(2, "Search Appointment")
        print_menu_item(3, "Exit")

def choose_role():
    clear_screen()
    print_header("WELCOME", CYAN)
    print("  Please select how you want to use PawSched.\n")
    print_menu_item(1, "Client")
    print_menu_item(2, "Owner / Staff")
    print_menu_item(3, "Exit")

    while True:
        choice = input("\n  Choose your role: ").strip()

        if choice == "1":
            return "Client"
        elif choice == "2":
            return "Owner/Staff"
        elif choice == "3":
            print("\n" + GREEN + "Thank you for using PawSched. Goodbye!" + RESET)
            raise SystemExit
        else:
            print(RED + "Invalid choice. Please enter 1, 2, or 3." + RESET)

def owner_staff_login():
    clear_screen()
    print_header("OWNER / STAFF LOGIN", MAGENTA)
    print("  Authorized personnel only.\n")

    username = input("  Username: ").strip()
    password = input("  Password: ").strip()

    if username == OWNER_USERNAME and password == OWNER_PASSWORD:
        print("\n" + GREEN + BOLD + "✓ Login successful!" + RESET)
        input("\n  Press Enter to continue...")
        return True

    print("\n" + RED + BOLD + "✗ Invalid username or password." + RESET)
    input("\n  Press Enter to return to login...")
    return False

# ---------------------------------------------------------
# MAIN PROGRAM
# ---------------------------------------------------------
while True:
    role = choose_role()

    if role == "Owner/Staff":
        if not owner_staff_login():
            continue

    while True:
        show_role_menu(role)
        choice = input("\n  Enter your choice: ").strip()

        if role == "Client":
            if choice == "1":
                add_appointment()
            elif choice == "2":
                reschedule_appointment()
            elif choice == "3":
                cancel_appointment()
            elif choice == "4":
                break   # go back to the role selection screen
            else:
                print(RED + "Invalid choice. Please enter a number from 1 to 4." + RESET)

            if choice in ("1", "2", "3"):
                input("\n  Press Enter to return to the menu...")

        elif role == "Owner/Staff":
            if choice == "1":
                view_appointments()
            elif choice == "2":
                search_appointment()
            elif choice == "3":
                break   # go back to the role selection screen
            else:
                print(RED + "Invalid choice. Please enter a number from 1 to 3." + RESET)

            if choice in ("1", "2"):
                input("\n  Press Enter to return to the menu...")
