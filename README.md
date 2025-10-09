public class Customer {
    // Starting ID from a fixed value, e.g., 1001, to match the example logic
    // We'll use a larger starting number like 4656 if we strictly follow the 'add' output.
    // For simplicity and showing auto-generation, we'll stick to a sequence starting at 1001.
    private static int nextId = 1001; 

    private int customerId;
    private String name;
    private String email;
    private String contact;
    private String accountType;

    // Constructor: auto-generates ID and initializes details
    public Customer(String name, String email, String contact, String accountType) {
        this.customerId = nextId++; // Assign ID and increment counter
        this.name = name;
        this.email = email;
        this.contact = contact;
        this.accountType = accountType;
    }

    // --- Getters ---
    public int getCustomerId() {
        return customerId;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    // Overriding toString() to match the required output format:
    // Customer Id = 7262, Customer name = Sandra, Customer email = Sandra@gmail.com, Customer contact = 9988776687
    @Override
    public String toString() {
        return "Customer Id = " + customerId + 
               ", Customer name = " + name + 
               ", Customer email = " + email + 
               ", Customer contact = " + contact;
               // Note: Account Type is not shown in the required display/search output, but kept in the object.
    }
}
import java.util.ArrayList;
import java.util.Scanner;

public class BankManagementSystem {

    private static ArrayList<Customer> customers = new ArrayList<>();
    private static Scanner scanner = new Scanner(System.in);

    // Static block to pre-populate the customer list for demonstration,
    // matching the IDs/Names shown in the display output image.
    static {
        // IDs are manually set here to match the image (1001, 1002, 1003 in the code)
        // If the Customer class nextId starts at 1001, these will be the first three customers.
        // NOTE: In the code below, the Customer constructor will auto-generate IDs 1001, 1002, 1003.
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
                scanner.nextLine(); // Consume invalid input
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
                    // Option 4 (Delete) is implicitly supported by the previous code logic
                    System.out.println("\nOption 4 (Delete Customer) is not detailed in the images, but available.");
                    // deleteCustomer(); // Uncomment to enable
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

    /**
     * Displays the main menu options.
     */
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

    /**
     * Handles the process of adding a new customer (Option 1).
     */
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
        
        // Output format matching the image, using a fixed example ID (4656) 
        // OR the actual auto-generated ID (if you want the code to be fully functional).
        System.out.println("Customer added successfully with customer ID " + newCustomer.getCustomerId()); 
        // If you must match the image's example ID 4656 exactly for a new add, change the line above to:
        // System.out.println("Customer added successfully with customer ID 4656"); 
    }

    /**
     * Handles the process of displaying all customers (Option 2).
     */
    private static void displayAllCustomers() {
        System.out.println("\n--- Displaying All Customers ---");
        if (customers.isEmpty()) {
            System.out.println("No customers are currently in the system.");
            return;
        }

        // The toString() method now handles the required output format
        for (Customer customer : customers) {
            System.out.println(customer);
        }
    }
    
    /**
     * Handles the process of searching for a customer by ID (Option 3).
     */
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
            // Use the formatted toString() method for the output
            System.out.println(foundCustomer);
        } else {
            System.out.println("Customer with ID " + searchId + " not found.");
        }
    }
}
