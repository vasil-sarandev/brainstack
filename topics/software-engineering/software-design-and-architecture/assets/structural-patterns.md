# Structural Design Patterns

#deep-dive

Structural design patterns explain how to assemble objects and classes into larger structures, while keeping these structures flexible and efficient.

---
## List of Structural Design Patterns

- [_Adapter_](#adapter) - Provides an interface between two unrelated entities so that they can work together.
- [_Composite_](#composite) - Used when we have to implement a part-whole hierarchy. For example, a diagram made of other pieces such as circle, square, triangle, etc.
- [_Proxy_](#proxy) - Provide a surrogate or placeholder for another object to control access to it.
- [_Flyweight_](#flyweight) - Caching and reusing object instances, used with immutable objects. For example, string pool.
- [_Facade_](#facade) - Creating a wrapper interfaces on top of existing interfaces to help client applications.
- [_Bridge_](#bridge) - The bridge design pattern is used to decouple the interfaces from implementation and hiding the implementation details from the client program.
- [_Decorator_](#decorator) - The decorator design pattern is used to modify the functionality of an object at runtime.

---
## Adapter

One of the great real life example of Adapter design pattern is mobile charger. Mobile battery needs 3 volts to charge but the normal socket produces either 120V (US) or 240V (India). So the mobile charger works as an adapter between mobile charging socket and the wall socket. We will try to implement multi-adapter using adapter design pattern in this tutorial. So first of all we will have two classes - Volt (to measure volts) and Socket (producing constant volts of 120V).

```Java
public class Volt {
	private int volts;
	public Volt(int v){
		this.volts=v;
	}
	public int getVolts() {
		return volts;
	}
	public void setVolts(int volts) {
		this.volts = volts;
	}
}
public class Socket {
	public Volt getVolt(){
		return new Volt(120);
	}
}
// adapter
public interface SocketAdapter {

	public Volt get120Volt();

	public Volt get12Volt();

	public Volt get3Volt();
}
// object adapter (Java Composition)
public class SocketObjectAdapterImpl implements SocketAdapter{

	//Using Composition for adapter pattern
	private Socket sock = new Socket();

	@Override
	public Volt get120Volt() {
		return sock.getVolt();
	}

	@Override
	public Volt get12Volt() {
		Volt v= sock.getVolt();
		return convertVolt(v,10);
	}

	@Override
	public Volt get3Volt() {
		Volt v= sock.getVolt();
		return convertVolt(v,40);
	}

	private Volt convertVolt(Volt v, int i) {
		return new Volt(v.getVolts()/i);
	}
}
// class adapter (Java Inheritance)
public class SocketClassAdapterImpl extends Socket implements SocketAdapter{

	@Override
	public Volt get120Volt() {
		return getVolt();
	}

	@Override
	public Volt get12Volt() {
		Volt v= getVolt();
		return convertVolt(v,10);
	}

	@Override
	public Volt get3Volt() {
		Volt v= getVolt();
		return convertVolt(v,40);
	}

	private Volt convertVolt(Volt v, int i) {
		return new Volt(v.getVolts()/i);
	}

}

```

---
## Composite

When we need to create a structure in a way that the objects in the structure has to be treated the same way, we can apply composite design pattern. Lets understand it with a real life example - A diagram is a structure that consists of Objects such as Circle, Lines, Triangle etc. When we fill the drawing with color (say Red), the same color also gets applied to the Objects in the drawing. Here drawing is made up of different parts and they all have same operations. Composite Pattern consists of following objects.
1. **Base Component** - Base component is the interface for all objects in the composition, client program uses base component to work with the objects in the composition. It can be an interface or an abstract class with some methods common to all the objects.
2. **Leaf** - Defines the behaviour for the elements in the composition. It is the building block for the composition and implements base component. It doesn’t have references to other Components.
3. **Composite** - It consists of leaf elements and implements the operations in base component.

```Java
// base componnet
public interface Shape {
	public void draw(String fillColor);
}

// leaf
public class Triangle implements Shape {

	@Override
	public void draw(String fillColor) {
		System.out.println("Drawing Triangle with color "+fillColor);
	}

}

// leaf
public class Circle implements Shape {

	@Override
	public void draw(String fillColor) {
		System.out.println("Drawing Circle with color "+fillColor);
	}

}

// composite
public class Drawing implements Shape{
	//collection of Shapes
	private List<Shape> shapes = new ArrayList<Shape>();

	@Override
	public void draw(String fillColor) {
		for(Shape sh : shapes)
		{
			sh.draw(fillColor);
		}
	}

	//adding shape to drawing
	public void add(Shape s){
		this.shapes.add(s);
	}
}

```

---
## Proxy

Proxy design pattern intent is: Provide a surrogate or placeholder for another object to control access to it.

The definition itself is very clear and proxy design pattern is used when we want to provide controlled access of a functionality.

```Java
public interface CommandExecutor {
	public void runCommand(String cmd) throws Exception;
}

public class CommandExecutorImpl implements CommandExecutor {
	@Override
	public void runCommand(String cmd) throws IOException {
        //some heavy implementation
		Runtime.getRuntime().exec(cmd);
		System.out.println("'" + cmd + "' command executed.");
	}

}

public class CommandExecutorProxy implements CommandExecutor {

	private boolean isAdmin;
	private CommandExecutor executor;

	public CommandExecutorProxy(String user, String pwd){
		if("Pankaj".equals(user) && "J@urnalD$v".equals(pwd)) isAdmin=true;
		executor = new CommandExecutorImpl();
	}

	@Override
	public void runCommand(String cmd) throws Exception {
		if(isAdmin){
			executor.runCommand(cmd);
		}else{
			if(cmd.trim().startsWith("rm")){
				throw new Exception("rm command is not allowed for non-admin users.");
			}else{
				executor.runCommand(cmd);
			}
		}
	}

}

```

---
## Flyweight

Caching and reusing object instances, used with immutable objects. For example, string pool.

Flyweight design pattern is used when we need to create a lot of Objects of a class. Since every object consumes memory space that can be crucial for low memory devices, such as mobile devices or embedded systems, flyweight design pattern can be applied to reduce the load on memory by sharing objects.

```JAVA
public interface Shape {
	public void draw(Graphics g, int x, int y, int width, int height,
			Color color);
}
public class Line implements Shape {
	@Override
	public void draw(Graphics line, int x1, int y1, int x2, int y2,
			Color color) {
		line.setColor(color);
		line.drawLine(x1, y1, x2, y2);
	}
}
public class Oval implements Shape {
	//intrinsic property
	private boolean fill;
	public Oval(boolean f){
		this.fill=f;
	}
	@Override
	public void draw(Graphics circle, int x, int y, int width, int height,
			Color color) {
		circle.setColor(color);
		circle.drawOval(x, y, width, height);
		if(fill){
			circle.fillOval(x, y, width, height);
		}
	}
}
// flyweight factory where the object instances are cached internally
public class FlyweightShapeFactory {

    // here the object instances are stored
	private static final HashMap<ShapeType,Shape> shapes = new HashMap<ShapeType,Shape>();

	public static Shape getShape(ShapeType type) {
		Shape shapeImpl = shapes.get(type);

		if (shapeImpl == null) {
			if (type.equals(ShapeType.OVAL_FILL)) {
				shapeImpl = new Oval(true);
			} else if (type.equals(ShapeType.OVAL_NOFILL)) {
				shapeImpl = new Oval(false);
			} else if (type.equals(ShapeType.LINE)) {
				shapeImpl = new Line();
			}
			shapes.put(type, shapeImpl);
		}
		return shapeImpl;
	}

	public static enum ShapeType{
		OVAL_FILL,OVAL_NOFILL,LINE;
	}
}
```

---
## Facade

Facade design pattern is used to help client applications to easily interact with the system.

Suppose we have an application with set of interfaces to use MySql/Oracle database and to generate different types of reports, such as HTML report, PDF report etc. So we will have different set of interfaces to work with different types of database. Now a client application can use these interfaces to get the required database connection and generate reports. But when the complexity increases or the interface behavior names are confusing, client application will find it difficult to manage it. So we can apply Facade design pattern here and provide a wrapper interface on top of the existing interface to help client application.

```JAVA
import java.sql.Connection;

public class MySqlHelper {
    public static Connection getMySqlDBConnection() {
        // Simulate getting MySQL DB connection
        System.out.println("Connecting to MySQL...");
        return null;
    }
}

public class OracleHelper {
    public static Connection getOracleDBConnection() {
        // Simulate getting Oracle DB connection
        System.out.println("Connecting to Oracle...");
        return null;
    }
}

// Facade class
public class HelperFacade {
    public static Connection getDBConnection(DBTypes dbType) {
        switch (dbType) {
            case MYSQL:
                return MySqlHelper.getMySqlDBConnection();
            case ORACLE:
                return OracleHelper.getOracleDBConnection();
            default:
                throw new IllegalArgumentException("Unsupported DB type");
        }
    }

    public enum DBTypes {
        MYSQL, ORACLE;
    }
}
```

---
## Bridge
The bridge design pattern is used to decouple the interfaces from implementation and hiding the implementation details from the client program.

```JAVA
public interface Color {
	public void applyColor();
}
public class RedColor implements Color{
	public void applyColor(){
		System.out.println("red.");
	}
}

public abstract class Shape {
	//Composition - implementor
	protected Color color;
	//constructor with implementor as input argument
	public Shape(Color c){
		this.color=c;
	}
    // this is the bridge abstract
	abstract public void applyColor();
}

public class Triangle extends Shape{
	public Triangle(Color c) {
		super(c);
	}

    // implementation for the bridge
	@Override
	public void applyColor() {
		System.out.print("Triangle filled with color ");
		color.applyColor();
	}

}
```

---
## Decorator

Decorator design pattern is used to modify the functionality of an object at runtime. At the same time other instances of the same class will not be affected by this, so individual object gets the modified behavior.

```JAVA
public interface Car {
	public void assemble();
}
public class BasicCar implements Car {

	@Override
	public void assemble() {
		System.out.print("Basic Car.");
	}

}
public class CarDecorator implements Car {

	protected Car car;

	public CarDecorator(Car c){
		this.car=c;
	}

	@Override
	public void assemble() {
		this.car.assemble();
	}

}


public class SportsCar extends CarDecorator {

	public SportsCar(Car c) {
		super(c);
	}

	@Override
	public void assemble(){
		super.assemble();
		System.out.print(" Adding features of Sports Car.");
	}
}


public static void main(String[] args) {
		Car sportsCar = new SportsCar(new BasicCar());
		sportsCar.assemble();
        //Basic Car. Adding features of Sports Car.
	}


```
