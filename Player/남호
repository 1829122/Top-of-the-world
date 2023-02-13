/*
이 프로그램은 다음 기능을 가지고 있습니다.
• 캐릭터는 플레이어의 조작에 의해 평면 방향으로 이동한다.
• 캐릭터의 이동 방향은 카메라가 향하고 있는 방향(=화면의 전방)을 기준으로 한다.
• 캐릭터는 다른 Collidor(충돌 판정물)를 피한다.즉 벽이나 기둥 속으로 파고들지 않는다.
• 캐릭터는 자신의 Collidor가 다른 Collidor에서 수직 방향으로 떨어지면 다른 Collidor에 접촉할 때까지 낙하한다.
• 캐릭터는 낙하 중 이동 방향은 변경할 수 없다.
• 캐릭터는 일정 각도의 Collidor 또는 일정량의 단차가 있는 Collidor가 있을 때 오를 수 있다.
• 캐릭터가 멈춰 있을 때 진행 방향은 즉시 변경할 수 있지만 이동 중인 경우 서서히 진행 방향이 바뀐다.
• 이동키가 눌려 바로 보행이 되고, 계속 눌려 일정 시간이 지나면 주행으로 변화한다.
• 이동 키가 떨어지면 캐릭터 이동은 멈춘다.
• 이동 스틱의 기울기에 따라 캐릭터는 정지, 보행, 주행으로 변화한다.
• 캐릭터는 정지, 보행, 주행, 낙하 상태에 맞춰 모션을 변화시킨다.
*/
using UnityEngine;
using System.Collections;

public class BasicCharCont : MonoBehaviour {
    private float walkSpeed = 2.0f; // 보행 속도
    private float runSpeed = 4.0f; // 주행 속도
    private float jumpForce = 3.0f;
    private Vector3 movement;   // 이동하는 벡터
    private KeyCode jumpKeyCode = KeyCode.Space;
    private float gravity = 20.0f; // 캐릭터로의 중력
    private float speedSmoothing = 10.0f;   // 回頭するときの滑らかさ
    private float rotateSpeed = 500.0f; // 回頭の速度
    private float runAfterSeconds = 0.1f;   // 走り終わったとき止まるまでの時間(秒)
   

    private Vector3 moveDirection = Vector3.zero;   // 커런트 이동 방향
    private float verticalSpeed = 0.0f;    // カレント垂直方向速度
    private float moveSpeed = 0.0f;    // カレント水平方向速度

    private CollisionFlags collisionFlags;    //  controller.Move가 반환하는 콜리전 플래그: 캐릭터가 무언가에 부딪혔을 때 사용

    private float walkTimeStart = 0.0f;    // 歩き始める速度

    // Use this for initialization
    void Start () {
        moveDirection = transform.TransformDirection(Vector3.forward);  // 캐릭터의 이동 방향을 캐릭터가 향하고 있는 방향으로 설정하다
      

    }


    // Update is called once per frame
    void Update()
    {
        Transform cameraTransform = Camera.main.transform;  // 카메라가 향하고 있는 방향을 얻다
        Vector3 forward = cameraTransform.TransformDirection(Vector3.forward);  // camera의 x-z 평면에서 forward 벡터를 구하다
        forward.y = 0;  // Y方向は無視：キャラは水平面しか移動しないため
        forward = forward.normalized;   // 方向を正規化する
        Vector3 right = new Vector3(forward.z, 0, -forward.x);    // 右方向ベクターは常にforwardに直交

        float v = Input.GetAxisRaw("Vertical"); // マウスもしくはコントローラスティックの垂直方向の値
        float h = Input.GetAxisRaw("Horizontal");   // マウスもしくはコントローラスティックの水平方向の値

        Vector3 targetDirection = h * right + v * forward;  // カメラと連動した進行方向を計算：視点の向きが前方方向

        if ((collisionFlags & CollisionFlags.CollidedBelow) != 0)   // キャラは接地しているか？：宙に浮いていないとき
        {
            if (targetDirection != Vector3.zero) // キャラは順方向を向いていないか？：つまり回頭している場合
            {
                if (moveSpeed < walkSpeed * 0.9) // ゆっくり移動か？
                {
                    moveDirection = targetDirection.normalized; // 止まっているときは即時ターン
                }
                else  // 移動しているときはスムースにターン
                {
                    moveDirection = Vector3.RotateTowards(moveDirection, targetDirection, rotateSpeed * Mathf.Deg2Rad * Time.deltaTime, 1000);
                    moveDirection = moveDirection.normalized;
                }
            }

            float curSmooth = speedSmoothing * Time.deltaTime;     // 방향을 스무스하게 변경
            float targetSpeed = Mathf.Min(targetDirection.magnitude, 1.0f); // 최소한의 속도를 설정

            // 걷는 속도와 달리는 속도의 전환: 처음에는 걷고 시간이 지나면 달린다
            if (Time.time - runAfterSeconds > walkTimeStart)
                targetSpeed *= runSpeed;
            else
                targetSpeed *= walkSpeed;

            // 移動速度を設定
            moveSpeed = Mathf.Lerp(moveSpeed, targetSpeed, curSmooth);

            Animator animator = GetComponent<Animator>();   // Animator コンポーネントを得る
            animator.SetFloat("spd", moveSpeed); // Animator に移動速度のパラメータを渡す
            animator.SetBool("fall", false);    // Animator に落下フラグのパラメータを渡す：落下していない

            if (moveSpeed < walkSpeed * 0.3)    // まだ歩きはじめ
                walkTimeStart = Time.time;  // その時間を保存しておく

            verticalSpeed = 0.0f;   // 垂直方向の速度をゼロに設定
        }
        else // 宙に浮いている
        {
            verticalSpeed -= gravity * Time.deltaTime;  // 중력을 적응
            if (verticalSpeed < -4.0)   // 떨어지는 속도가 일정을 넘으면
            {
                Animator animator = GetComponent<Animator>();   // Animator コンポーネントを得る
                animator.SetBool("fall", true);   // Animator に落下フラグのパラメータを渡す：落下している
            }
        }

        movement = moveDirection * moveSpeed + new Vector3(0, verticalSpeed, 0);   // キャラの移動量を計算
        movement *= Time.deltaTime;

        CharacterController controller = GetComponent<CharacterController>();   // キャラクターコントローラコンポーネントを取得
        collisionFlags = controller.Move(movement);   // 캐릭터를 이동을 캐릭터 컨트롤러에 전달하다

        if ((collisionFlags & CollisionFlags.CollidedBelow) != 0)   // 공중에 떠 있지 않은 경우
        {
            transform.rotation = Quaternion.LookRotation(moveDirection);       //이동 방향으로 회두: 떠 있으면 회두하지 않는다
        }
        /*if (controller.isGrounded == true)
        {
            if (Input.GetKey(jumpKeyCode))
            {
                
            }
        }*/
        
    }
}
