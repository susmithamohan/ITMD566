# SERVER CODE:
/* Java project done by Susmitha Mohan, Priyanka Jagadesh Kumar, Vaishali Pachisiya
 * This is Java Project that allows Multiple Clients to access a single Server
 * This Server Class allows Multiple clients to request for Mathematical operations
 * And returns the result to the respective clients
 * It also returns the number of clients using the server if requested
 */

import java.io.IOException;
import java.net.Socket;
import java.net.ServerSocket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;


/**
 *
 * @author Susmitha Mohan, Priyanka Jagadesh Kumar, Vaishali Pachisiya
 */
public class Server {

    public static int a = 0;//Creating Static variable

	/**
     * @param args the command line arguments
     */
    public static void main(String[] args) {
    	
        int portNumber = Integer.parseInt("8080"); //creating port number
        ExecutorService executor = null; //Initializing ExecutiveService variable
        try (ServerSocket serverSocket = new ServerSocket(portNumber);){
            executor = Executors.newFixedThreadPool(5); //creating a limit of clients up to 5 threads
            System.out.println("Waiting for client");
            while(true){
                Socket clientSocket = serverSocket.accept();//Establishing Client connection in to the socket             
                Runnable worker = new RequestHandler(clientSocket);//Starting up runnable interface 
                executor.execute(worker);      
                }
            
        } catch (IOException e) {//Exception handling
            System.out.println("Exception caught when trying to listen on port" + portNumber+"or listening for a connection");
            System.out.println(e.getMessage());
        } finally {
            if (executor != null){
                executor.shutdown();
            }
        }
    }
    
}

### Request Handling Class:

/* Java project done by Susmitha Mohan, Priyanka Jagadesh Kumar, Vaishali Pachisiya
 * This is Java Project that allows Multiple Clients to access a single Server
 * This Request Handling Class performs the Mathematical operations in the Server class
 * And sends the result to the Client
 */

import java.net.Socket;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.BufferedWriter;
import java.net.ServerSocket;


/**
 *
 * @author Susmitha Mohan, Priyanka Jagadesh Kumar, Vaishali Pachisiya
 */
public class RequestHandler implements Runnable {    
    private final Socket client; //Creating object for Socket
    ServerSocket serverSocket = null;

    public RequestHandler(Socket client) {
        this.client = client;//Constructor
    }

    @Override
    public void run() {
    	
    	//System.out.println(Server.a);    	
    	int b=++Server.a;//Incrementing the Static Variable
    	//System.out.println(b);
        try{
        	
            System.out.println("Please enter Operation (A for Add, S for Sub,M for Mult,D for Div, 'MO' for Mod, 'c 0 0'count) and the two operands for Mathematical operations or type end");
            BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
            BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(client.getOutputStream()));
            System.out.println("Thread Started with name:" + Thread.currentThread().getName());
            String userInput;
            while((userInput = in.readLine()) != null){
                userInput = userInput.replaceAll("[^A-Za-z0-9 ]","");//Eliminating symbols in user input
                System.out.println("Recieved message from"+Thread.currentThread().getName()+":"+userInput);
                if (userInput.startsWith("end")){
                    System.out.println("Server Closed");
                    serverSocket.close();//closing the socket connection
                } 
                else{
                	
                  System.out.println("Entering else");  
                  String[] arrayInput= userInput.split(" ");//splitting the user input into array objects
                  String operator = arrayInput[0].trim();//storing first array object as an operator                
                  int operand1 = Integer.parseInt(arrayInput[1].trim());// storing second array object as operand 1
                  int operand2 = Integer.parseInt(arrayInput[2].trim());// storing third array object as operand 2
                  System.out.println(operand1);System.out.println(operand2);//for reference
                  
                  int result = 0;
                  //execution of operations using conditional statement
                  switch (operator) {
                    case "A":
                        result = operand1 + operand2;//addition operation
                        break;
                    case "S":
                        System.out.println("Entering SUB");
                        result = operand1 - operand2;//subtraction
                        break;
                    case "M":
                        result = operand1 * operand2;//multiplication
                        break;
                    case "D":
                        result = operand1 / operand2;//division
                    case "MO":
                    	result = operand1 % operand2;//modulus
                        break;
                    case "c":
                    	
                    	result=b;//Displaying number of clients connected to the server
                    	break;
                                       
                    default:
                        System.out.println("invalid operand: " + userInput);
                        writer.flush();
                
            }
                  System.out.println("Send back result: " + result);
                  writer.write(result);
                }
                writer.newLine();
                writer.flush();
            }
            
        } catch (IOException e) {
            System.out.println("I/O exception: " + e);
        } catch(Exception ex){
            System.out.println("Exception in thread Run. Exception: "+ex);
            
        }
    }
    
}

# CLIENT CODE

/* Java project done by Susmitha Mohan, Priyanka Jagadesh Kumar, Vaishali Pachisiya
 * This is Java Project that allows Multiple Clients to access a single Server
 * This Client Class requests the Server class for Mathematical operations
 * And gets the result from the Server class
 * It can also know the number of clients using the server if requested
 */
package client;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;
import java.net.UnknownHostException;


/**
 *
 * @author Susmitha Mohan, Priyanka Jagadesh Kumar, Vaishali Pachisiya
 */
public class Client {

	/**
	 * @param args the command line arguments
	 */
	public static void main(String[] args) {
		String hostName = "localhost"; // creating local host

		int portNumber = Integer.parseInt("8080"); //Assigning same port number as Server
		try {
			Socket socket = new Socket(hostName, portNumber);  //Establishing socket connection          
			PrintWriter out = new PrintWriter(socket.getOutputStream(), true);//creating output variable
			BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));//creating input variable
			BufferedReader stdIn = new BufferedReader (new InputStreamReader(System.in));//getting user input
			String userInput;
			while ((userInput = stdIn.readLine()) != null){
				out.println(userInput);
				System.out.println("Send to Server"+userInput);// Displaying input entered by user
				System.out.println("Result: "+in.read());//Displaying result from Server
				
			}
		}catch (UnknownHostException e) {// Exception Handling
			System.err.println("Don't know about host" + hostName);
			System.exit(1);
		} catch (IOException ex) {
			System.out.println("Couldn't get I/O for the connection to" + hostName);
			System.exit(1);
		} 
	}

}
