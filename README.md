resource "aws_route53_zone" "example_com" {
  name = "example_com.com"  # Replace with your domain name
}
data "aws_instance" "tetris_game_instance" {
  instance_id = aws_instance.tetris_game_instance.id
}
resource "aws_security_group" "tetris_game_sg" {
  name        = "tetris-game-sg"
  description = "Security group for the Tetris game"

  vpc_id = "vpc-"  # Replace with your VPC ID

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Allow traffic from anywhere
  }
}

resource "aws_instance" "tetris_game_instance" {
  ami           = "ami-0fa399d9c130ec923"  # Replace with the desired AMI ID
  instance_type = "t2.micro"  # Choose an appropriate instance type

  tags = {
    Name = "tetris-game-instance"
  }

  security_groups = [aws_security_group.tetris_game_sg.name]

  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd

    # Configure and start the Apache web server
    systemctl start httpd
    systemctl enable httpd

    # Download and deploy the Tetris game code
    git clone https://github.com/yourusername/tetris-game.git /var/www/html

    # Additional configuration for your game (e.g., database setup, environment variables, etc.)
    # ...

    # Restart Apache to apply changes
    systemctl restart httpd
    EOF
}
resource "aws_route53_record" "tetris_game_dns" {
  zone_id = aws_route53_zone.example_com.zone_id
  name    = "tetris"  # Your desired subdomain
  type    = "A"
  ttl     = "300"
  records = [data.aws_instance.tetris_game_instance.public_ip]
 
 
}
