import java.util.ArrayList;
import java.util.Scanner;

public class BankManagementSystem {

    // --- Nested Customer Class ---
    private static class Customer {
        // Static counter for auto-generating unique Customer IDs
        private static int nextId = 1001; 

        private int customerId;
        private String name;
        private String email;
        private String contact;
        private String accountType; // Not shown in output, but stored

        // Constructor: auto-generates ID and initializes details
        public Customer(String name, String email, String contact, String accountType) {
            this.customerId = nextId++; 
            this.name = name;
            this.email = email;
            this.contact = contact;
            this.accountType = accountType;
        }

        public int getCustomerId() {
            return customerId;
        }

        // Overriding toString() to match the required output format:
        // Customer Id = 1001, Customer name = Sandra, Customer email = Sandra@gmail.com, Customer contact = 9988776687
        @Override
        public String toString() {
            return "Customer Id = " + customerId + 
                   ", Customer name = " + name + 
                   ", Customer email = " + email + 
                   ", Customer contact = " + contact;
        }
    }
    // -----------------------------


    private static ArrayList<Customer> customers = new ArrayList<>();
    private static Scanner scanner = new Scanner(System.in);

    // Static block to pre-populate data matching the display output image.
    static {
        customers.add(new Customer("Sandra", "Sandra@gmail.com", "9988776687", "Savings"));
        customers.add(new Customer("Michelle", "Michelle@gmail.com", "9876543210", "Current"));
        customers.add(new Customer("Steve", "Steve@gmail.com", "1122334455", "Savings"));
    }


    public static void main(String[] args) {
        int choice;

        do {
            displayMenu();
            if (scanner.hasNextInt()) {
                choice = scanner.nextInt();
                scanner.nextLine(); // Consume newline
            } else {
                System.out.println("\n--- Invalid input. Please enter a number from the menu. ---");
                scanner.nextLine(); 
                choice = 0; 
            }

            switch (choice) {
                case 1:
                    addNewCustomer();
                    break;
                case 2:
                    displayAllCustomers();
                    break;
                case 3:
                    searchCustomer();
                    break;
                case 4:
                    deleteCustomer(); // Included for completeness
                    break; 
                case 5:
                    System.out.println("\n--- Exiting the bank application. Goodbye! 👋 ---");
                    break;
                default:
                    System.out.println("\n--- Invalid choice. Please enter a number between 1 and 5. ---");
            }
        } while (choice != 5);

        scanner.close(); 
    }

    private static void displayMenu() {
        System.out.println("\n=============================================");
        System.out.println("Welcome to Standard Chartered Bank");
        System.out.println("Please enter your choice:");
        System.out.println("1 for Add new Customer");
        System.out.println("2 for Display Customers");
        System.out.println("3 for Search Customer");
        System.out.println("4 for Delete Customer");
        System.out.println("5 for Exit the bank application");
        System.out.print("Enter your choice: ");
    }

    private static void addNewCustomer() {
        System.out.println("\nPlease enter customer details:");
        
        System.out.print("Enter name: ");
        String name = scanner.nextLine();

        System.out.print("Enter email: ");
        String email = scanner.nextLine();
        
        System.out.print("Enter contact: ");
        String contact = scanner.nextLine();

        System.out.print("Enter account type (Savings or Current): ");
        String accountType = scanner.nextLine();
        
        Customer newCustomer = new Customer(name, email, contact, accountType);
        customers.add(newCustomer);
        
        System.out.println("Customer added successfully with customer ID " + newCustomer.customerId); 
    }

    private static void displayAllCustomers() {
        System.out.println("\n--- Displaying All Customers ---");
        if (customers.isEmpty()) {
            System.out.println("No customers are currently in the system.");
            return;
        }
        for (Customer customer : customers) {
            System.out.println(customer);
        }
    }
    
    private static void searchCustomer() {
        System.out.println("\n--- Search Customer ---");
        System.out.print("Please enter customer id: ");
        
        if (!scanner.hasNextInt()) {
            System.out.println("Invalid ID format. Please enter a number.");
            scanner.nextLine(); 
            return;
        }
        int searchId = scanner.nextInt();
        scanner.nextLine(); 

        Customer foundCustomer = null;
        for (Customer customer : customers) {
            if (customer.getCustomerId() == searchId) {
                foundCustomer = customer;
                break;
            }
        }

        if (foundCustomer != null) {
            System.out.println(foundCustomer);
        } else {
            System.out.println("Customer with ID " + searchId + " not found.");
        }
    }
    
    private static void deleteCustomer() {
        System.out.println("\n--- Delete Customer ---");
        System.out.print("Enter Customer ID to delete: ");
        
        if (!scanner.hasNextInt()) {
            System.out.println("Invalid ID format. Please enter a number.");
            scanner.nextLine(); 
            return;
        }
        int deleteId = scanner.nextInt();
        scanner.nextLine(); 
        
        boolean removed = customers.removeIf(customer -> customer.getCustomerId() == deleteId);

        if (removed) {
            System.out.println("Customer with ID " + deleteId + " has been successfully deleted.");
        } else {
            System.out.println("Customer with ID " + deleteId + " not found. Deletion failed.");
        }
    }
}
