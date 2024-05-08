using System;
using System.Net;
using System.Net.Mail;

public interface IEmailValidator
{
    bool IsValidEmailFormat(string email);
    bool IsDomainActive(string email);
    bool IsMailServerResponding(string email);
}

public class EmailValidator : IEmailValidator
{
    public bool IsValidEmailFormat(string email)
    {
        try
        {
            MailAddress mailAddress = new MailAddress(email);
            return true;
        }
        catch (FormatException)
        {
            return false;
        }
    }

    public bool IsDomainActive(string email)
    {
        try
        {
            string domain = email.Split('@')[1];
            IPHostEntry hostEntry = Dns.GetHostEntry(domain);
            return true;
        }
        catch (Exception)
        {
            return false;
        }
    }

    public bool IsMailServerResponding(string email)
    {
        try
        {
            string domain = email.Split('@')[1];
            using (var client = new SmtpClient(domain))
            {
                client.Port = 25;
                client.Timeout = 5000;
                client.Send(new MailMessage());
                return true;
            }
        }
        catch (Exception)
        {
            return false;
        }
    }
}

[TestFixture]
public class EmailValidatorTests
{
    private IEmailValidator validator;

    [SetUp]
    public void SetUp()
    {
        validator = new EmailValidator();
    }

    [Test]
    public void IsValidEmailFormat_ValidEmail_ReturnsTrue()
    {
        // Arrange
        string email = "example@example.com";

        // Act
        bool result = validator.IsValidEmailFormat(email);

        // Assert
        Assert.IsTrue(result);
    }

    [Test]
    public void IsValidEmailFormat_InvalidEmail_ReturnsFalse()
    {
        // Arrange
        string email = "invalidemail";

        // Act
        bool result = validator.IsValidEmailFormat(email);

        // Assert
        Assert.IsFalse(result);
    }

    [Test]
    public void IsDomainActive_ActiveDomain_ReturnsTrue()
    {
        // Arrange
        string email = "example@example.com";

        // Act
        bool result = validator.IsDomainActive(email);

        // Assert
        Assert.IsTrue(result);
    }

    [Test]
    public void IsDomainActive_InactiveDomain_ReturnsFalse()
    {
        // Arrange
        string email = "example@nonexistentdomain.com";

        // Act
        bool result = validator.IsDomainActive(email);

        // Assert
        Assert.IsFalse(result);
    }

    [Test]
    public void IsMailServerResponding_ServerResponding_ReturnsTrue()
    {
        // Arrange
        string email = "example@example.com";

        // Act
        bool result = validator.IsMailServerResponding(email);

        // Assert
        Assert.IsTrue(result);
    }

    [Test]
    public void IsMailServerResponding_ServerNotResponding_ReturnsFalse()
    {
        // Arrange
        string email = "example@nonexistentdomain.com";

        // Act
        bool result = validator.IsMailServerResponding(email);

        // Assert
        Assert.IsFalse(result);
    }
}
