# Behavioral Design Patterns

This type of design patterns provide solution for the better interaction between objects, how to provide lose coupling, and flexibility to extend easily in future.

---
## List of Behavioral Design Patterns 

- [_Template Method_](#template-method) - used to create a template method stub and defer some of the steps of implementation to the subclasses.
- [_Mediator_](#mediator) - used to provide a centralized communication medium between different objects in a system.
- [_Chain of Responsibility_](#chain-of-responsibility) - used to achieve loose coupling in software design where a request from the client is passed to a chain of objects to process them.
- [_Observer_](#observer) - useful when you are interested in the state of an object and want to get notified whenever there is any change.
- [_Strategy_](#strategy) - Strategy pattern is used when we have multiple algorithm for a specific task and client decides the actual implementation to be used at runtime.
- [_Command_](#command) - Command Pattern is used to implement lose coupling in a request-response model.
- [_State_](#state) - State design pattern is used when an Object change it’s behavior based on it’s internal state.
- [_Visitor_](#visitor) - Visitor pattern is used when we have to perform an operation on a group of similar kind of Objects.
- [_Interpreter_](#interpreter) - defines a grammatical representation for a language and provides an interpreter to deal with this grammar.
- [_Iterator_](#iterator) - used to provide a standard way to traverse through a group of Objects.
- [_Memento_](#memento) - The memento design pattern is used when we want to save the state of an object so that we can restore later on.

---
## Template Method

Used to create a template method stub and defer some of the steps of implementation to the subclasses.

Template method defines the steps to execute an algorithm and it can provide default implementation that might be common for all or some of the subclasses.

```Java
public abstract class HouseTemplate {
	//template method, final so subclasses can't override.
	public final void buildHouse(){
		buildFoundation();
		buildPillars();
		buildWalls();
		buildWindows();
		System.out.println("House is built.");
	}

	//default implementation
	private void buildWindows() {
		System.out.println("Building Glass Windows");
	}

	//methods to be implemented by subclasses
	public abstract void buildWalls();
	public abstract void buildPillars();

	private void buildFoundation() {
		System.out.println("Building foundation with cement,iron rods and sand");
	}
}

public class WoodenHouse extends HouseTemplate {

	@Override
	public void buildWalls() {
		System.out.println("Building Wooden Walls");
	}

	@Override
	public void buildPillars() {
		System.out.println("Building Pillars with Wood coating");
	}

}

public class GlassHouse extends HouseTemplate {

	@Override
	public void buildWalls() {
		System.out.println("Building Glass Walls");
	}

	@Override
	public void buildPillars() {
		System.out.println("Building Pillars with glass coating");
	}

}
```

---
## Mediator

Mediator design pattern is used to provide a centralized communication medium between different objects in a system.

Mediator design pattern is very helpful in an enterprise application where multiple objects are interacting with each other. If the objects interact with each other directly, the system components are tightly-coupled with each other that makes higher maintainability cost and not hard to extend. Mediator pattern focuses on provide a mediator between objects for communication and help in implementing lose-coupling between objects.

```JAVA
public interface ChatMediator {

	public void sendMessage(String msg, User user);

	void addUser(User user);
}

public abstract class User {
	protected ChatMediator mediator;
	protected String name;

	public User(ChatMediator med, String name){
		this.mediator=med;
		this.name=name;
	}

	public abstract void send(String msg);

	public abstract void receive(String msg);
}

public class ChatMediatorImpl implements ChatMediator {

	private List<User> users;

	public ChatMediatorImpl(){
		this.users=new ArrayList<>();
	}

	@Override
	public void addUser(User user){
		this.users.add(user);
	}

	@Override
	public void sendMessage(String msg, User user) {
		for(User u : this.users){
			//message should not be received by the user sending it
			if(u != user){
				u.receive(msg);
			}
		}
	}

}

public class UserImpl extends User {

	public UserImpl(ChatMediator med, String name) {
		super(med, name);
	}

	@Override
	public void send(String msg){
		System.out.println(this.name+": Sending Message="+msg);
		mediator.sendMessage(msg, this);
	}
	@Override
	public void receive(String msg) {
		System.out.println(this.name+": Received Message:"+msg);
	}

}

public class ChatClient {

	public static void main(String[] args) {
		ChatMediator mediator = new ChatMediatorImpl();
		User user1 = new UserImpl(mediator, "Pankaj");
		User user2 = new UserImpl(mediator, "Lisa");=
		mediator.addUser(user1);
		mediator.addUser(user2);

		user1.send("Hi All");

	}

}

```

---
## Chain of Responsibility

Chain of responsibility pattern is used to achieve loose coupling in software design where a request from client is passed to a chain of objects to process them. Then the object in the chain will decide themselves who will be processing the request and whether the request is required to be sent to the next object in the chain or not.

```JAVA

public class Currency {

	private int amount;

	public Currency(int amt){
		this.amount=amt;
	}

	public int getAmount(){
		return this.amount;
	}
}

public interface DispenseChain {

	void setNextChain(DispenseChain nextChain);

	void dispense(Currency cur);
}

public class Dollar50Dispenser implements DispenseChain {

	private DispenseChain chain;

	@Override
	public void setNextChain(DispenseChain nextChain) {
		this.chain=nextChain;
	}

	@Override
	public void dispense(Currency cur) {
		if(cur.getAmount() >= 50){
			int num = cur.getAmount()/50;
			int remainder = cur.getAmount() % 50;
			System.out.println("Dispensing "+num+" 50$ note");
			if(remainder !=0) this.chain.dispense(new Currency(remainder));
		}else{
			this.chain.dispense(cur);
		}
	}

}

public class Dollar20Dispenser implements DispenseChain{

	private DispenseChain chain;

	@Override
	public void setNextChain(DispenseChain nextChain) {
		this.chain=nextChain;
	}

	@Override
	public void dispense(Currency cur) {
		if(cur.getAmount() >= 20){
			int num = cur.getAmount()/20;
			int remainder = cur.getAmount() % 20;
			System.out.println("Dispensing "+num+" 20$ note");
			if(remainder !=0) this.chain.dispense(new Currency(remainder));
		}else{
			this.chain.dispense(cur);
		}
	}

}

public class ATMDispenseChain {

	private DispenseChain c1;

	public ATMDispenseChain() {
		// initialize the chain
		this.c1 = new Dollar50Dispenser();
		DispenseChain c2 = new Dollar20Dispenser();
		DispenseChain c3 = new Dollar10Dispenser();

		// set the chain of responsibility
		c1.setNextChain(c2);
		c2.setNextChain(c3);
	}

	public static void main(String[] args) {
		ATMDispenseChain atmDispenser = new ATMDispenseChain();
		while (true) {
			int amount = 0;
			System.out.println("Enter amount to dispense");
			Scanner input = new Scanner(System.in);
			amount = input.nextInt();
			if (amount % 10 != 0) {
				System.out.println("Amount should be in multiple of 10s.");
				return;
			}
			// process the request
			atmDispenser.c1.dispense(new Currency(amount));
		}

	}

}

// Enter amount to dispense
// 530
// Dispensing 10 50$ note
// Dispensing 1 20$ note
// Dispensing 1 10$ note
// Enter amount to dispense

```

---
## Observer

Observer Pattern is one of the behavioral design pattern. Observer design pattern is useful when you are interested in the state of an object and want to get notified whenever there is any change. In observer pattern, the object that watch on the state of another object are called Observer and the object that is being watched is called Subject.

```JAVA
public interface Subject {

	//methods to register and unregister observers
	public void register(Observer obj);
	public void unregister(Observer obj);

	//method to notify observers of change
	public void notifyObservers();

	//method to get updates from subject
	public Object getUpdate(Observer obj);

}


public interface Observer {

	//method to update the observer, used by subject
	public void update();

	//attach with subject to observe
	public void setSubject(Subject sub);
}

public class MyTopic implements Subject {

	private List<Observer> observers;
	private String message;
	private boolean changed;
	private final Object MUTEX= new Object();

	public MyTopic(){
		this.observers=new ArrayList<>();
	}
	@Override
	public void register(Observer obj) {
		if(obj == null) throw new NullPointerException("Null Observer");
		synchronized (MUTEX) {
		if(!observers.contains(obj)) observers.add(obj);
		}
	}

	@Override
	public void unregister(Observer obj) {
		synchronized (MUTEX) {
		observers.remove(obj);
		}
	}

	@Override
	public void notifyObservers() {
		List<Observer> observersLocal = null;
		//synchronization is used to make sure any observer registered after message is received is not notified
		synchronized (MUTEX) {
			if (!changed)
				return;
			observersLocal = new ArrayList<>(this.observers);
			this.changed=false;
		}
		for (Observer obj : observersLocal) {
			obj.update();
		}

	}

	@Override
	public Object getUpdate(Observer obj) {
		return this.message;
	}

	//method to post message to the topic
	public void postMessage(String msg){
		System.out.println("Message Posted to Topic:"+msg);
		this.message=msg;
		this.changed=true;
		notifyObservers();
	}

}


public class MyTopicSubscriber implements Observer {

	private String name;
	private Subject topic;

	public MyTopicSubscriber(String nm){
		this.name=nm;
	}

	@Override
	public void update() {
		String msg = (String) topic.getUpdate(this);
		if(msg == null){
			System.out.println(name+":: No new message");
		}else
		System.out.println(name+":: Consuming message::"+msg);
	}

	@Override
	public void setSubject(Subject sub) {
		this.topic=sub;
	}

}

public class ObserverPatternTest {

	public static void main(String[] args) {
		//create subject
		MyTopic topic = new MyTopic();

		//create observers
		Observer obj1 = new MyTopicSubscriber("Obj1");
		Observer obj2 = new MyTopicSubscriber("Obj2");
		Observer obj3 = new MyTopicSubscriber("Obj3");

		//register observers to the subject
		topic.register(obj1);
		topic.register(obj2);
		topic.register(obj3);

		//attach observer to subject
		obj1.setSubject(topic);
		obj2.setSubject(topic);
		obj3.setSubject(topic);

		//check if any update is available
		obj1.update();

		//now send message to subject
		topic.postMessage("New Message");
	}

}

// Obj1:: No new message
// Message Posted to Topic:New Message
// Obj1:: Consuming message::New Message
// Obj2:: Consuming message::New Message
// Obj3:: Consuming message::New Message
```

---
## Strategy

Strategy pattern is also known as Policy Pattern. We define multiple algorithms and let client application pass the algorithm to be used as a parameter. One of the best example of strategy pattern is Collections.sort() method that takes Comparator parameter. Based on the different implementations of Comparator interfaces, the Objects are getting sorted in different ways.

```JAVA
public interface PaymentStrategy {

    public void pay(int amount);

}


public class CreditCardStrategy implements PaymentStrategy {

	private String name;
	private String cardNumber;
	private String cvv;
	private String dateOfExpiry;

	public CreditCardStrategy(String nm, String ccNum, String cvv, String expiryDate){
		this.name=nm;
		this.cardNumber=ccNum;
		this.cvv=cvv;
		this.dateOfExpiry=expiryDate;
	}
	@Override
	public void pay(int amount) {
		System.out.println(amount +" paid with credit/debit card");
	}

}


public class PaypalStrategy implements PaymentStrategy {

	private String emailId;
	private String password;

	public PaypalStrategy(String email, String pwd){
		this.emailId=email;
		this.password=pwd;
	}

	@Override
	public void pay(int amount) {
		System.out.println(amount + " paid using Paypal.");
	}

}

public class Item {

	private String upcCode;
	private int price;

	public Item(String upc, int cost){
		this.upcCode=upc;
		this.price=cost;
	}

	public String getUpcCode() {
		return upcCode;
	}

	public int getPrice() {
		return price;
	}

}

public class ShoppingCart {

	//List of items
	List<Item> items;

	public ShoppingCart(){
		this.items=new ArrayList<Item>();
	}

	public void addItem(Item item){
		this.items.add(item);
	}

	public void removeItem(Item item){
		this.items.remove(item);
	}

	public int calculateTotal(){
		int sum = 0;
		for(Item item : items){
			sum += item.getPrice();
		}
		return sum;
	}

	public void pay(PaymentStrategy paymentMethod){
		int amount = calculateTotal();
		paymentMethod.pay(amount);
	}
}

public class ShoppingCartTest {

	public static void main(String[] args) {
		ShoppingCart cart = new ShoppingCart();

		Item item1 = new Item("1234",10);
		Item item2 = new Item("5678",40);

		cart.addItem(item1);
		cart.addItem(item2);

		//pay by paypal
		cart.pay(new PaypalStrategy("myemail@example.com", "mypwd"));

		//pay by credit card
		cart.pay(new CreditCardStrategy("Pankaj Kumar", "1234567890123456", "786", "12/15"));
	}

}
```

---
## Command

In command pattern, the request is send to the invoker and invoker pass it to the encapsulated command object. Command object passes the request to the appropriate method of Receiver to perform the specific action. The client program create the receiver object and then attach it to the Command. Then it creates the invoker object and attach the command object to perform an action. Now when client program executes the action, it’s processed based on the command and receiver object.

---
## State

If we have to change the behavior of an object based on its state, we can have a state variable in the Object. Then use if-else condition block to perform different actions based on the state. State design pattern is used to provide a systematic and loosely coupled way to achieve this through Context and State implementations.

State Pattern Context is the class that has a State reference to one of the concrete implementations of the State. Context forwards the request to the state object for processing.

**Basic example with if/else**

```JAVA
public class TVRemoteBasic {

	private String state="";

	public void setState(String state){
		this.state=state;
	}

	public void doAction(){
		if(state.equalsIgnoreCase("ON")){
			System.out.println("TV is turned ON");
		}else if(state.equalsIgnoreCase("OFF")){
			System.out.println("TV is turned OFF");
		}
	}

	public static void main(String args[]){
		TVRemoteBasic remote = new TVRemoteBasic();

		remote.setState("ON");
		remote.doAction();

		remote.setState("OFF");
		remote.doAction();
	}

}
```

**Example with Context**

```JAVA

public interface State {

	public void doAction();
}
public class TVStartState implements State {

	@Override
	public void doAction() {
		System.out.println("TV is turned ON");
	}

}
public class TVStopState implements State {

	@Override
	public void doAction() {
		System.out.println("TV is turned OFF");
	}

}
public class TVContext implements State {

	private State tvState;

	public void setState(State state) {
		this.tvState=state;
	}

	public State getState() {
		return this.tvState;
	}

	@Override
	public void doAction() {
		this.tvState.doAction();
	}

}

public class TVRemote {

	public static void main(String[] args) {
		TVContext context = new TVContext();
		State tvStartState = new TVStartState();
		State tvStopState = new TVStopState();

		context.setState(tvStartState);
		context.doAction();


		context.setState(tvStopState);
		context.doAction();

	}

}
```

---
## Visitor

Visitor pattern is used when we have to perform an operation on a group of similar kind of Objects. With the help of visitor pattern, we can move the operational logic from the objects to another class.

```JAVA
public interface ItemElement {

	public int accept(ShoppingCartVisitor visitor);
}

public class Book implements ItemElement {

	private int price;
	private String isbnNumber;

	public Book(int cost, String isbn){
		this.price=cost;
		this.isbnNumber=isbn;
	}

	public int getPrice() {
		return price;
	}

	public String getIsbnNumber() {
		return isbnNumber;
	}

	@Override
	public int accept(ShoppingCartVisitor visitor) {
		return visitor.visit(this);
	}

}

public class Fruit implements ItemElement {

	private int pricePerKg;
	private int weight;
	private String name;

	public Fruit(int priceKg, int wt, String nm){
		this.pricePerKg=priceKg;
		this.weight=wt;
		this.name = nm;
	}

	public int getPricePerKg() {
		return pricePerKg;
	}


	public int getWeight() {
		return weight;
	}

	public String getName(){
		return this.name;
	}

	@Override
	public int accept(ShoppingCartVisitor visitor) {
		return visitor.visit(this);
	}

}


public interface ShoppingCartVisitor {

	int visit(Book book);
	int visit(Fruit fruit);
}

public class ShoppingCartVisitorImpl implements ShoppingCartVisitor {

	@Override
	public int visit(Book book) {
		int cost=0;
		//apply 5$ discount if book price is greater than 50
		if(book.getPrice() > 50){
			cost = book.getPrice()-5;
		}else cost = book.getPrice();
		System.out.println("Book ISBN::"+book.getIsbnNumber() + " cost ="+cost);
		return cost;
	}

	@Override
	public int visit(Fruit fruit) {
		int cost = fruit.getPricePerKg()*fruit.getWeight();
		System.out.println(fruit.getName() + " cost = "+cost);
		return cost;
	}

}

public class ShoppingCartClient {

	public static void main(String[] args) {
		ItemElement[] items = new ItemElement[]{new Book(20, "1234"),new Book(100, "5678"),
				new Fruit(10, 2, "Banana"), new Fruit(5, 5, "Apple")};

		int total = calculatePrice(items);
		System.out.println("Total Cost = "+total);
	}

	private static int calculatePrice(ItemElement[] items) {
		ShoppingCartVisitor visitor = new ShoppingCartVisitorImpl();
		int sum=0;
		for(ItemElement item : items){
			sum = sum + item.accept(visitor);
		}
		return sum;
	}

}

```

---
## Interpreter

Defines a grammatical representation for a language and provides an interpreter to deal with this grammar.

```JAVA
class Context {
    // Any global information needed for interpretation
}

interface Expression {
    int interpret(Context context);
}

class NumberExpression implements Expression {
    private int number;

    public NumberExpression(int number) {
        this.number = number;
    }

    @Override
    public int interpret(Context context) {
        return number;
    }
}

class AdditionExpression implements Expression {
    private Expression left;
    private Expression right;

    public AdditionExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret(Context context) {
        return left.interpret(context) + right.interpret(context);
    }
}

class MultiplicationExpression implements Expression {
    private Expression left;
    private Expression right;

    public MultiplicationExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret(Context context) {
        return left.interpret(context) * right.interpret(context);
    }
}

class Interpreter {
    private Context context;

    public Interpreter(Context context) {
        this.context = context;
    }

    public int interpret(String expression) {
        // Parse expression and create expression tree
        Expression expressionTree = buildExpressionTree(expression);

        // Interpret expression tree
        return expressionTree.interpret(context);
    }

    private Expression buildExpressionTree(String expression) {
        // Logic to parse expression and create expression tree
        // For simplicity, assume the expression is already parsed
        // and represented as an expression tree
        return new AdditionExpression(
            new NumberExpression(2),
            new MultiplicationExpression(
                new NumberExpression(3),
                new NumberExpression(4)
            )
        );
    }
}

public class Client {
    public static void main(String[] args) {
        // Input expression
        String expression = "2 + 3 * 4";

        // Create interpreter
        Context context = new Context();
        Interpreter interpreter = new Interpreter(context);

        // Interpret expression
        int result = interpreter.interpret(expression);
        System.out.println("Result: " + result);
    }
}
```

---
## Iterator

Iterator design pattern in one of the behavioral pattern. Iterator pattern is used to provide a standard way to traverse through a group of Objects. Iterator pattern is widely used in Java Collection Framework. Iterator interface provides methods for traversing through a collection.

```JAVA
// We could also create an iterator ourselves. Java has iterators built in for some types.
public class Geeks {
    public static void main(String[] args) {

        // Create an ArrayList
        // and add some elements
        ArrayList<String> al = new ArrayList<>();
        al.add("A");
        al.add("B");
        al.add("C");

        // Obtain an iterator for the ArrayList
        Iterator<String> it = al.iterator();

        // Iterate through the elements
        // and print each one
        while (it.hasNext()) {

            // Get the next element
            String n = it.next();
            System.out.println(n);
        }
    }
}
```

---
## Memento

The Memento Design Pattern offers a solution to implement undoable actions. We can do this by saving the state of an object at a given instant and restoring it if the actions performed since need to be undone.

Practically, the object whose state needs to be saved is called an Originator. The Caretaker is the object triggering the save and restore of the state, which is called the Memento.

```JAVA
import java.util.ArrayList;
import java.util.List;

// Originator
class Document {
    private String content;

    public Document(String content) {
        this.content = content;
    }

    public void write(String text) {
        this.content += text;
    }

    public String getContent() {
        return this.content;
    }

    public DocumentMemento createMemento() {
        return new DocumentMemento(this.content);
    }

    public void restoreFromMemento(DocumentMemento memento) {
        this.content = memento.getSavedContent();
    }
}

// Memento
class DocumentMemento {
    private String content;

    public DocumentMemento(String content) {
        this.content = content;
    }

    public String getSavedContent() {
        return this.content;
    }
}

// Caretaker
class History {
    private List<DocumentMemento> mementos;

    public History() {
        this.mementos = new ArrayList<>();
    }

    public void addMemento(DocumentMemento memento) {
        this.mementos.add(memento);
    }

    public DocumentMemento getMemento(int index) {
        return this.mementos.get(index);
    }
}

public class Main {
    public static void main(String[] args) {
        Document document = new Document("Initial content\n");
        History history = new History();

        // Write some content
        document.write("Additional content\n");
        history.addMemento(document.createMemento());

        // Write more content
        document.write("More content\n");
        history.addMemento(document.createMemento());

        // Restore to previous state
        document.restoreFromMemento(history.getMemento(1));

        // Print document content
        System.out.println(document.getContent());
    }
}
```
