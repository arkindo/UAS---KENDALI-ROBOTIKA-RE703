# UAS---KENDALI-ROBOTIKA-RE703
AXCEEL ARKINDO 4222201057 &amp; DAVA SAMUDRA 4222201045


METODE FSM 
import rclpy
from rclpy.node import Node
from geometry_msgs.msg import Twist
from sensor_msgs.msg import LaserScan

class TurtleBotFSM(Node):
    def __init__(self):
        super().__init__('turtlebot_fsm')
        self.publisher = self.create_publisher(Twist, '/cmd_vel', 10)
        self.subscription = self.create_subscription(LaserScan, '/scan', self.scan_callback, 10)
        
        # Inisialisasi State
        self.state = "FORWARD" 
        self.obstacle_distance = 1.0

    def scan_callback(self, msg):
        # Mengambil data jarak depan (index 0)
        self.obstacle_distance = msg.ranges[0]
        self.execute_fsm()

    def execute_fsm(self):
        msg = Twist()

        if self.state == "FORWARD":
            if self.obstacle_distance < 0.5:
                self.state = "ROTATE"
                self.get_logger().info("Hambatan terdeteksi! Berputar...")
            else:
                msg.linear.x = 0.2  # Maju
                msg.angular.z = 0.0

        elif self.state == "ROTATE":
            if self.obstacle_distance > 0.8:
                self.state = "FORWARD"
                self.get_logger().info("Jalan aman! Maju lagi...")
            else:
                msg.linear.x = 0.0
                msg.angular.z = 0.5  # Berputar

        self.publisher.publish(msg)

def main():
    rclpy.init()
    node = TurtleBotFSM()
    rclpy.spin(node)
    rclpy.shutdown()
