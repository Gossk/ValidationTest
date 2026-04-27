# ValidationTest

## Appointment-Scheduling

### Appointment-Service

```csharp
namespace Appointment_Scheduling;

public interface IRepository
{
    void Save(string petname);
}

public class AppointmentService
{
    private readonly IRepository _repo;

    public AppointmentService(IRepository repo)
    {
        _repo = repo;
    }

    public bool CanSchedule(int current, int max)
    {
        return current < max;
    }

    public bool Schedule(string petname, int current, int max)
    {
        if (string.IsNullOrWhiteSpace(petname))
            return false;
        if (current >= max)
            return false;

        _repo.Save(petname);
        return true;
    }
}
```

### FakeRepository

```csharp
namespace Appointment_Scheduling;

public class FakeRepository : IRepository
{
    public List<string> SavedAppointments = new List<string>();

    public void Save(string petname)
    {
        SavedAppointments.Add(petname);
    }
}
```

### Schedule-Appointment-Tests

```csharp
using Appointment_Scheduling;
using Xunit;
using Moq;
using Assert = Xunit.Assert;

namespace Schedule_Appointment;

public class Tests
{
    [Fact]
    public void ShouldReturnTrue_WhenThereIsSpace()
    {
        var mockRepo = new Mock<IRepository>();
        var service = new AppointmentService(mockRepo.Object);

        var result = service.CanSchedule(2, 5);

        Assert.True(result);
    }

    [Fact]
    public void ShouldReturnFalse_WhenScheduleIsFull()
    {
        var mockRepo = new Mock<IRepository>();
        var service = new AppointmentService(mockRepo.Object);

        var result = service.Schedule("Shaylo", 5, 5);

        Assert.False(result);
    }

    [Fact]
    public void ShouldReturnToRepository_WhenTheAppointmentIsValid()
    {
        var mockRepo = new Mock<IRepository>();
        var service = new AppointmentService(mockRepo.Object);

        service.Schedule("Shaylo", 2, 5);

        mockRepo.Verify(r => r.Save("Shaylo"), Times.Once);
    }

    [Fact]
    public void ShouldSaveAppointment_WhenUsingRealRepository()
    {
        var repo = new FakeRepository();
        var service = new AppointmentService(repo);

        var result = service.Schedule("Shaylo", 2, 5);

        Assert.True(result);
        Assert.Contains("Shaylo", repo.SavedAppointments);
    }
}
```

### Caso Veterinaria

* **Actor:** Recepcionista
* **Acción:** Programar cita
* **Restricciones:** horario lleno, mascota sin nombre, doctor no disponible
* **Resultado positivo:** cita creada, cita guardada

---

## Gym-System

### Reservation-Service

```csharp
namespace Gym_System;

public interface IRepository
{
    void Save(string classId);
}

public class ReservationService
{
    private readonly IRepository _repo;

    public ReservationService(IRepository repo)
    {
        _repo = repo;
    }

    public bool Reserve(string classId, int current, int max)
    {
        if (current >= max)
            return false;

        _repo.Save(classId);
        return true;
    }

    public bool CanReserve(int current, int max)
    {
        return current < max;
    }
}
```

### FakeRepository

```csharp
namespace Gym_System;

public class FakeRepository : IRepository
{
    public List<string> SavedReservations = new List<string>();

    public void Save(string classId)
    {
        SavedReservations.Add(classId);
    }
}
```

### ReservationServiceTests

```csharp
using Xunit;
using Moq;

namespace Gym_System.Tests;

public class ReservationServiceTests
{
    [Fact]
    public void ShouldReturnTrue_WhenThereIsSpace()
    {
        var mockRepo = new Mock<IRepository>();
        var service = new ReservationService(mockRepo.Object);

        var result = service.CanReserve(3, 10);

        Xunit.Assert.True(result);
    }

    [Fact]
    public void ShouldCallRepository_WhenClassIsFull()
    {
        var mockRepo = new Mock<IRepository>();
        var service = new ReservationService(mockRepo.Object);

        var result = service.Reserve("A", 10, 10);

        Xunit.Assert.False(result);
    }

    [Fact]
    public void ShouldCallRepository_WhenReservationIsSuccessful()
    {
        var mockRepo = new Mock<IRepository>();
        var service = new ReservationService(mockRepo.Object);

        service.Reserve("A1", 2, 10);

        mockRepo.Verify(r => r.Save("A1"), Times.Once);
    }

    [Fact]
    public void ShouldSaveReservation_WhenUsingRealRepository()
    {
        var repo = new FakeRepository();
        var service = new ReservationService(repo);

        var result = service.Reserve("A1", 2, 10);

        Xunit.Assert.True(result);
        Xunit.Assert.Contains("A1", repo.SavedReservations);
    }
}
```
