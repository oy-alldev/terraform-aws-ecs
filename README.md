이 Terraform 모듈은 AWS의 ECS(Elastic Container Service) 리소스를 생성하는 데 사용됩니다. 아래는 주요 기능과 사용법을 정리한 번역입니다.

📦 주요 기능
ECS 클러스터 생성 (Fargate 또는 EC2 오토스케일링 사용 가능)

ECS 서비스 생성 (태스크 정의, 컨테이너 정의 등 포함)

클러스터 및 서비스 리소스를 통합 모듈 또는 하위 모듈로 구성 가능

📄 디자인 문서 참조

🧑‍💻 통합 모듈 예시 사용법
hcl
복사
편집
module "ecs" {
  source = "terraform-aws-modules/ecs/aws"

  cluster_name = "ecs-integrated"

  # 클러스터 설정
  cluster_configuration = {
    execute_command_configuration = {
      logging = "OVERRIDE"
      log_configuration = {
        cloud_watch_log_group_name = "/aws/ecs/aws-ec2"
      }
    }
  }

  # Fargate 및 Spot 설정
  fargate_capacity_providers = {
    FARGATE = { default_capacity_provider_strategy = { weight = 50 } }
    FARGATE_SPOT = { default_capacity_provider_strategy = { weight = 50 } }
  }

  # 서비스 정의
  services = {
    ecsdemo-frontend = {
      cpu    = 1024
      memory = 4096

      container_definitions = {
        fluent-bit = {
          image     = "fluent-bit 이미지"
          cpu       = 512
          memory    = 1024
          essential = true
          firelens_configuration = {
            type = "fluentbit"
          }
        }
        ecs-sample = {
          image     = "ecs-sample 이미지"
          cpu       = 512
          memory    = 1024
          port_mappings = [{ containerPort = 80 }]
        }
      }

      load_balancer = {
        service = {
          target_group_arn = "TG ARN"
          container_name   = "ecs-sample"
          container_port   = 80
        }
      }

      subnet_ids = ["subnet-abc", "subnet-def"]
      security_group_rules = {
        alb_ingress_3000 = {
          type        = "ingress"
          from_port   = 80
          to_port     = 80
          protocol    = "tcp"
        }
        egress_all = {
          type        = "egress"
          from_port   = 0
          to_port     = 0
          protocol    = "-1"
          cidr_blocks = ["0.0.0.0/0"]
        }
      }
    }
  }

  tags = {
    Environment = "Development"
    Project     = "Example"
  }
}
📁 예시
ECS 클러스터 전체 예시

EC2 오토스케일링 예시

Fargate 사용 예시

🛠️ 입력 변수 (일부)
이름	설명	기본값
cluster_name	ECS 클러스터 이름	""
create	리소스 생성 여부 (전체 제어용)	true
fargate_capacity_providers	Fargate 용량 설정	{}
services	생성할 서비스들 정의	{}
tags	태그 맵	{}

📤 출력값 (일부)
이름	설명
cluster_arn	클러스터 ARN
services	생성된 서비스들의 속성 맵
task_exec_iam_role_arn	태스크 실행 역할 ARN

👨‍👩‍👧‍👦 기여자
이 모듈은 Anton Babenko 및 여러 기여자들에 의해 관리됩니다.

⚖️ 라이선스
Apache-2.0 라이선스. LICENSE 보기
