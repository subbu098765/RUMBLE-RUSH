using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class CH : MonoBehaviour
{
    [SerializeField] float lLD = 2f;
    [SerializeField] AudioClip s;
    [SerializeField] AudioClip c;
    [SerializeField] ParticleSystem sucess;
    [SerializeField] ParticleSystem crashp;
    AudioSource aS;
    bool isTransitioning = false;

    void Start()
    {
        aS = GetComponent<AudioSource>();
    }

    void OnCollisionEnter(Collision other)
    {
        if (isTransitioning) { return; }

        switch (other.gameObject.tag)
        {
            case "Friendly":
                Debug.Log("good to see u Dear");
                break;
            case "Finish":
                Debug.Log("you have Done It Manh");
                StartSucess();
                break;
            case "fuel":
                Debug.Log("U Gained Ur Fuel man just go for it");
                break;
            default:
                StartChecks();
                break;
        }
    }

    void StartSucess()
    {
        isTransitioning = true;
        aS.Stop();
        aS.PlayOneShot(s);
        sucess.Play();
        GetComponent<Movement>().enabled = false;
        Invoke("LoadNextLevel", lLD);
    }

    void StartChecks()
    {
        isTransitioning = true;
        aS.Stop();
        aS.PlayOneShot(c);
        crashp.Play();
        GetComponent<Movement>().enabled = false;
        Invoke("Reloadlevel", lLD);
    }

    void LoadNextLevel()
    {
        int currentSceneIndex = SceneManager.GetActiveScene().buildIndex;
        int nextSceneIndex = currentSceneIndex + 1;
        if (nextSceneIndex == SceneManager.sceneCountInBuildSettings)
        {
            nextSceneIndex = 0;
        }
        SceneManager.LoadScene(nextSceneIndex);
    }

    void Reloadlevel()
    {
        int currentSceneIndex = SceneManager.GetActiveScene().buildIndex;
        SceneManager.LoadScene(currentSceneIndex);
    }
}

public class Movement : MonoBehaviour
{
    [SerializeField] float mainThrust = 1000f;
    [SerializeField] float rThrust = 100f;
    [SerializeField] AudioClip mainEngine;
    [SerializeField] ParticleSystem mainEngineParticles;
    [SerializeField] ParticleSystem sTp;
    [SerializeField] ParticleSystem LSp;
    Rigidbody rBody;
    AudioSource As;

    void Start()
    {
        rBody = GetComponent<Rigidbody>();
        As = GetComponent<AudioSource>();
    }

    void Update()
    {
        processthrust();
        processRotation();
    }

    void processthrust()
    {
        if (Input.GetKey(KeyCode.Space))
        {
            StartThrusting();
        }
        else
        {
            stopsThrusting();
        }
    }

    void processRotation()
    {
        if (Input.GetKey(KeyCode.A))
        {
            leftRotate();
        }
        else if (Input.GetKey(KeyCode.D))
        {
            rightRotate();
        }
        else
        {
            StopRotates();
        }
    }

    void stopsThrusting()
    {
        As.Stop();
        mainEngineParticles.Stop();
    }

    void StartThrusting()
    {
        rBody.AddRelativeForce(Vector3.up * mainThrust * Time.deltaTime);

        if (!As.isPlaying)
        {
            As.PlayOneShot(mainEngine);
        }
        if (!mainEngineParticles.isPlaying)
        {
            mainEngineParticles.Play();
        }
    }

    void StopRotates()
    {
        sTp.Stop();
        LSp.Stop();
    }

    void rightRotate()
    {
        ApplyRotation(-rThrust);
        if (!sTp.isPlaying)
        {
            sTp.Play();
        }
    }

    void leftRotate()
    {
        ApplyRotation(rThrust);
        if (!LSp.isPlaying)
        {
            LSp.Play();
        }
    }

    void ApplyRotation(float rThrust)
    {
        rBody.freezeRotation = true;
        transform.Rotate(Vector3.forward * rThrust * Time.deltaTime);
        rBody.freezeRotation = false;
    }
}

public class osc : MonoBehaviour
{
    Vector3 StartingPause;
    [SerializeField] Vector3 movementVector;
    [SerializeField][Range(0, 1)] float movementFactor;
    [SerializeField] float Period = 3f;

    void Start()
    {
        StartingPause = transform.position;
        Debug.Log(StartingPause);
    }

    void Update()
    {
        rotations();
    }

    private void rotations()
    {
        if (Period <= Mathf.Epsilon) { return; }
        float Cycles = Time.time / Period;
        const float Tau = Mathf.PI * 2;
        float rawSinewave = Mathf.Sin(Cycles * Tau);

        Vector3 offset = movementVector * movementFactor;
        transform.position = StartingPause + offset;
        movementFactor = (rawSinewave + 1f) / 2f;
    }
}

